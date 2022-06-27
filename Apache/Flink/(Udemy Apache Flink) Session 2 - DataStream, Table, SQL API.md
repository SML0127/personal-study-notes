# Section 1: DataStream, Table, SQL API 

## DataSream API

 - Data Sources for DataStream API
   - readTextFile(Path) - read line by line and returns them as strings
   - readFile(fileInputFormat, path)
   - readFile(fileInputFormat, path, watchType, interval, pathFilter)
     - interval: 주기적으로 새로운 data가 file에 append 됬는지 확인 후 read
     - watchType: PROCESS_CONTINUOUSLY, PROCESS_ONCE
     - pathFilter: used to exclude some files

   - SocketTextStream
     - Reads data from a socket. Elements can be separated by a delimiter

   - addSource
     - To add a custom data source outside of Flink (e.g. kafka)


 - Data Sinks for DataStream api
   - writeAsText() / TextOutputFormat
   - writeAsCSV()
   - print()
   - writeUsingOutputFormat() / FileOutputFormat
   - writeToSocket
   - addSink
     - To add a custom data sink outside of Flink (e.g. kafka, Flume, hdfs)


 - Flink의 reading process는 크게 2개의 task로 구분
   - Monitoring → 주어진 path 모니터링, split, 할당
     - Single, non-parallel
     - Scan path based on watchType
     - Divide into splits
     - Assign splits to readers

   - Actual Reading → 실제 read
     - Performed by multiple readers
     - Readers run parallelly
     - Each split read by only 1 reader 

 - Wordcount & DataStreaming 예제
   - Java
   ```` java
   public class WordCountStreaming {
     public static void main(String[] args) throws Exception {
       // set up the stream execution environment
       final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

       // Same with batch example
       final ParameterTool params = ParameterTool.fromArgs(args);

       // Same with batch example
       env.getConfig().setGlobalJobParameters(params);

       // data source = socket
       DataStream < String > text = env.socketTextStream("localhost", 9999);

       DataStream < Tuple2 < String, Integer >> counts = text.filter(new FilterFunction < String > () {
           public boolean filter(String value) {
             return value.startsWith("N");
           }
         })
         .map(new Tokenizer()) // split up the lines in pairs (2-tuples) containing: tuple2 {(name,1)...}
         .keyBy(t -> t.f0) // groupBy for Table API, keyBy for DataStream
         .sum(1); // group by the tuple field "0" and sum up tuple field "1"

       // data sink = print() 
       counts.print();

       // execute program
       env.execute("Streaming WordCount");
     }

     public static final class Tokenizer implements MapFunction < String, Tuple2 < String, Integer >> {
       public Tuple2 < String, Integer > map(String value) {
         return new Tuple2 < String, Integer > (value, Integer.valueOf(1));
       }
     }
   }
   ````
   - Python
   ````python
   import argparse
   import logging
   import sys

   from pyflink.common import WatermarkStrategy, Encoder, Types
   from pyflink.datastream import StreamExecutionEnvironment, RuntimeExecutionMode
   from pyflink.datastream.connectors import (FileSource, StreamFormat, FileSink, OutputFileConfig, RollingPolicy)

   def word_count(input_path, output_path):
       env = StreamExecutionEnvironment.get_execution_environment()

       # datastream인데 mode를 batch로 (why? file = bounded data)
       env.set_runtime_mode(RuntimeExecutionMode.BATCH)
       # write all the data to one file
       env.set_parallelism(1)

       # define the source
       if input_path is not None:
           ds = env.from_source(
               source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
               watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
               source_name="file_source"
           )
       else:
           print("Executing word_count example with default input data set.")
           print("Use --input to specify file input.")
           return

       def split(line):
           yield from line.split()


       # compute word count
       ds = ds.flat_map(split) \
           .map(lambda i: (i, 1), output_type=Types.TUPLE([Types.STRING(), Types.INT()])) \
           .key_by(lambda i: i[0]) \
           .reduce(lambda i, j: (i[0], i[1] + j[1]))


       # define the sink
       if output_path is not None:
           ds.sink_to(
               sink=FileSink.for_row_format(base_path=output_path, encoder=Encoder.simple_string_encoder())
               .with_output_file_config(OutputFileConfig.builder().with_part_prefix("prefix").with_part_suffix(".ext").build())
               .with_rolling_policy(RollingPolicy.default_rolling_policy())
               .build()
           )
       else:
           print("Printing result to stdout. Use --output to specify output path.")
           ds.print()

       # submit for execution
       env.execute()


   if __name__ == '__main__':
       logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
       parser = argparse.ArgumentParser()
       parser.add_argument('--input',dest='input',required=False,help='Input file to process.')
       parser.add_argument('--output',dest='output',required=False,help='Output file to write results to.')

       argv = sys.argv[1:]
       known_args, _ = parser.parse_known_args(argv)

       word_count(known_args.input, known_args.output)
   ````
 - Reduce operation
   - keyBy for DataStream API, groupBy for Table API
     - Java
     ```` java
     public class AverageProfit {
       public static void main(String[] args) throws Exception {
         // set up the streaming execution environment
         final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

         // data = ['01-06-2018,June,Category5,Bat,12', '', ...]
         DataStream < String > data = env.readTextFile("/shared/myuser/codes/reduce_operator/avg");

         // month, product, category, profit, count
         DataStream < Tuple5 < String, String, String, Integer, Integer >> mapped =
           data.map(new Splitter()); // tuple  [June,Category5,Bat,12,1]
         //        [June,Category4,Perfume,10,1]

         // groupBy 'month'
         DataStream < Tuple5 < String, String, String, Integer, Integer >> reduced = mapped.keyBy(t -> t.f0).reduce(new Reduce1());

         // June { [Category5,Bat,12,1] Category4,Perfume,10,1}	//rolling reduce
         // reduced = { [Category4,Perfume,22,2] ..... }
         // month, avg. profit
         DataStream < Tuple2 < String, Double >> profitPerMonth =
           reduced.map(new MapFunction < Tuple5 < String, String, String, Integer, Integer > , Tuple2 < String, Double >> () {
             public Tuple2 < String, Double > map(Tuple5 < String, String, String, Integer, Integer > input) {
               return new Tuple2 < String, Double > (input.f0, new Double((input.f3 * 1.0) / input.f4));
             }
           });

         //profitPerMonth.print();
         profitPerMonth.writeAsText("/shared/profit_per_month.txt");

         // execute program
         env.execute("Avg Profit Per Month");
       }

       // *************************************************************************
       // USER FUNCTIONS                                                                                  // pre_result  = Category4,Perfume,22,2
       // *************************************************************************

       public static class Reduce1 implements ReduceFunction < Tuple5 < String, String, String, Integer, Integer >> {
         public Tuple5 < String, String, String, Integer, Integer > reduce(Tuple5 < String, String, String, Integer, Integer > current, Tuple5 < String, String, String, Integer, Integer > pre_result) {
           return new Tuple5 < String, String, String, Integer, Integer > (current.f0, current.f1, current.f2, current.f3 + pre_result.f3, current.f4 + pre_result.f4);
         }
       }

       public static class Splitter implements MapFunction < String, Tuple5 < String, String, String, Integer, Integer >> {
         public Tuple5 < String, String, String, Integer, Integer > map(String value) // 01-06-2018,June,Category5,Bat,12
         {
           String[] words = value.split(","); // words = [{01-06-2018},{June},{Category5},{Bat}.{12}
           // ignore timestamp, we don't need it for any calculations
           return new Tuple5 < String, String, String, Integer, Integer > (words[1],
             words[2],
             words[3],
             Integer.parseInt(words[4]),
             1);
         } //    June    Category5      Bat                      12
       }
     }
     ````
     - Python
     ```` python
     import argparse
     import logging
     import sys

     from pyflink.common import WatermarkStrategy, Encoder, Types
     from pyflink.datastream import StreamExecutionEnvironment, RuntimeExecutionMode
     from pyflink.datastream.connectors import (FileSource, StreamFormat, FileSink, OutputFileConfig, RollingPolicy)
     from pyflink.datastream.tests.test_util import DataStreamTestSinkFunction

     def average_profit(input_path, output_path):
         env = StreamExecutionEnvironment.get_execution_environment()

         # datastream인데 mode를 batch로 (why? file = bounded data)
         env.set_runtime_mode(RuntimeExecutionMode.BATCH)
         # write all the data to one file
         env.set_parallelism(1)

         # ds = ['01-06-2018,June,Category5,Bat,12', '', ...]
         # define the source
         if input_path is not None:
             ds = env.from_source(
                 source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
                 watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
                 source_name="file_source"
             )
         else:
             print("Executing word_count example with default input data set.")
             print("Use --input to specify file input.")
             return


         # compute average profit per month
         ds = ds.map(lambda line: (line.split(",")[1], line.split(",")[2], line.split(",")[3], line.split(",")[4], 1), output_type=Types.TUPLE([Types.STRING(),Types.STRING(), Types.STRING(), Types.INT(). Types.INT()])) \
             .key_by(lambda i: i[0]) \ # group by month
             .reduce(lambda i, j: (i[0], i[3] + j[3], i[4] + j[4]))
             .map(lambda i: (i[0], float(i[1] / i[2])))

         # define the sink
         if output_path is not None:
             ds.sink_to(
                 sink=FileSink.for_row_format(base_path=output_path, encoder=Encoder.simple_string_encoder())
                 .with_output_file_config(OutputFileConfig.builder().with_part_prefix("prefix").with_part_suffix(".ext").build())
                 .with_rolling_policy(RollingPolicy.default_rolling_policy())
                 .build()
             )
         else:
             print("Printing result to stdout. Use --output to specify output path.")
             ds.print()

         # submit for execution
         env.execute()


     if __name__ == '__main__':
         logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
         parser = argparse.ArgumentParser()
         parser.add_argument('--input',dest='input',required=False,help='Input file to process.')
         parser.add_argument('--output',dest='output',required=False,help='Output file to write results to.')

         argv = sys.argv[1:]
         known_args, _ = parser.parse_known_args(argv) 
         average_profit(known_args.input, known_args.output)
     ````
 - Aggregation operation
   - Java
   ```` java
   public class Aggregation {
     public static void main(String[] args) throws Exception {
       // set up the streaming execution environment
       final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

       DataStream < String > data = env.readTextFile("/home/myuser/avg1");

       // month, category, product, profit, 
       DataStream < Tuple4 < String, String, String, Integer >> mapped = data.map(new Splitter()); // tuple  [June,Category5,Bat,12]
       //       [June,Category4,Perfume,10,1]
       mapped.keyBy(t -> t.f0).sum(3).writeAsText("/home/myuser/out1");

       // min(), max()는 최소/최대 값만 메모리에 유지, 그외의 field는 메모리 유지x
       mapped.keyBy(t -> t.f0).min(3).writeAsText("/home/myuser/out2");
       // minBy(), maxBy() 다른 filed도 메모리에 유지 
       mapped.keyBy(t -> t.f0).minBy(3).writeAsText("/home/myuser/out3");
       mapped.keyBy(t -> t.f0).max(3).writeAsText("/home/myuser/out4");
       mapped.keyBy(t -> t.f0).maxBy(3).writeAsText("/home/myuser/out5");
       // execute program
       env.execute("Aggregation");
     }


     public static class Splitter implements MapFunction < String, Tuple4 < String, String, String, Integer >> {
       public Tuple4 < String,
       String,
       String,
       Integer > map(String value) // 01-06-2018,June,Category5,Bat,12
       {
         String[] words = value.split(","); // words = [{01-06-2018},{June},{Category5},{Bat}.{12}
         // ignore timestamp, we don't need it for any calculations
         return new Tuple4 < String, String, String, Integer > (words[1], words[2], words[3], Integer.parseInt(words[4]));
       } //    June    Category5      Bat               12 
     }
   }
   ````
   - Python
   ```` pythnon
   import argparse
   import logging
   import sys

   from pyflink.common import WatermarkStrategy, Encoder, Types
   from pyflink.datastream import StreamExecutionEnvironment, RuntimeExecutionMode
   from pyflink.datastream.connectors import (FileSource, StreamFormat, FileSink, OutputFileConfig, RollingPolicy)

   def aggregation(input_path):
       env = StreamExecutionEnvironment.get_execution_environment()

       # datastream인데 mode를 batch로 (why? file = bounded data)
       env.set_runtime_mode(RuntimeExecutionMode.BATCH)
       # write all the data to one file
       env.set_parallelism(1)

       # define the source
       if input_path is not None:
           ds = env.from_source(
               source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
               watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
               source_name="file_source"
           )
       else:
           print("Executing word_count example with default input data set.")
           print("Use --input to specify file input.")
           return

       # ds = [June,Category4,Perfume,10,1]
       ds = ds.map(lambda line: (line.split(",")[1], line.split(",")[2], line.split(",")[3], int(line.split(",")[4])) 


       ds.key_by(lambda i: i[0]).min(3).sink_to(
               sink=FileSink.for_row_format(base_path="/output/path/min.txt", encoder=Encoder.simple_string_encoder())
               .with_rolling_policy(RollingPolicy.default_rolling_policy())
               .build()
           ) 
       ds.key_by(lambda i: i[0]).min_by(3).sink_to(
               sink=FileSink.for_row_format(base_path="/output/path/min_by.txt", encoder=Encoder.simple_string_encoder())
               .with_rolling_policy(RollingPolicy.default_rolling_policy())
               .build()
           )  
       ds.key_by(lambda i: i[0]).max(3).sink_to(
               sink=FileSink.for_row_format(base_path="/output/path/max.txt", encoder=Encoder.simple_string_encoder())
               .with_rolling_policy(RollingPolicy.default_rolling_policy())
               .build()
           )  
       ds.key_by(lambda i: i[0]).max_by(3).sink_to(
               sink=FileSink.for_row_format(base_path="/output/path/max_by.txt", encoder=Encoder.simple_string_encoder())
               .with_rolling_policy(RollingPolicy.default_rolling_policy())
               .build()
           ) 

       # submit for execution
       env.execute()


   if __name__ == '__main__':
       logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
       parser = argparse.ArgumentParser()
       parser.add_argument('--input',dest='input',required=False,help='Input file to process.')

       argv = sys.argv[1:]
       known_args, _ = parser.parse_known_args(argv)      
       aggregation(known_args.input)
   ````
   
 - Side outputs
   - Java
   ```` java
   public class SplitDemo {
     public static void main(String[] args) throws Exception
    {
       // set up the stream execution environment
       final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
       final ParameterTool params = ParameterTool.fromArgs(args);
       env.getConfig().setGlobalJobParameters(params);

       DataStream < String > text = env.readTextFile("/shared/myuser/codes/split_operator/oddeven");

       // String type side output for Even values
       final OutputTag < String > evenOutTag = new OutputTag < String > ("even-string-output") {};

       // Integer type side output for Odd values
       final OutputTag < Integer > oddOutTag = new OutputTag < Integer > ("odd-int-output") {};

       SingleOutputStreamOperator < Integer > mainStream = text
         .process(new ProcessFunction < String, Integer > () {
           @Override
           public void processElement(
             String value,
             Context ctx,
             Collector < Integer > out) throws Exception {

             int intVal = Integer.parseInt(value);

             // get all data in regular output as well
             out.collect(intVal);

             if (intVal % 2 == 0) {
               // emit data to side output for even output
               ctx.output(evenOutTag, String.valueOf(intVal));
             } else {
               // emit data to side output for even output
               ctx.output(oddOutTag, intVal);
             }
           }
         });

       DataStream < String > evenSideOutputStream = mainStream.getSideOutput(evenOutTag);
       DataStream < Integer > oddSideOutputStream = mainStream.getSideOutput(oddOutTag);

       evenSideOutputStream.writeAsText("/home/myuser/even");
       oddSideOutputStream.writeAsText("/home/myuser/odd");

       // execute program
       env.execute("ODD EVEN");
     }
   }
   ````
   - Python
   ```` pythnon
   import argparse
   import logging
   import sys

   from pyflink.common import WatermarkStrategy, Encoder, Types
   from pyflink.datastream import StreamExecutionEnvironment, RuntimeExecutionMode
   from pyflink.datastream.connectors import (FileSource, StreamFormat, FileSink, OutputFileConfig, RollingPolicy)
   from pyflink.datastream.tests.test_util import DataStreamTestSinkFunction


   def side_output(input_path):
       env = StreamExecutionEnvironment.get_execution_environment()
       env.set_runtime_mode(RuntimeExecutionMode.BATCH)
       env.set_parallelism(1)

       # define the source
       if input_path is not None:
           ds = env.from_source(
               source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
               watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
               source_name="file_source"
           )
       else:
           print("Executing word_count example with default input data set.")
           print("Use --input to specify file input.")
           return


       tag = OutputTag("side", Types.INT())  
       //tag2 = OutputTag("side2", Types.INT())

       class MyProcessfunction(ProcessFunction):
           def process_element(self, value, txt: "ProcessFunction.txt") : 
               if int(value[0]) % 2 == 0:
                   yield value[0]
               else:
                   yield tag, value[0]   
               #if int(value[0]) % 3 == 0:
               #    yield tag2, value[0]

       ds = ds.process(process_element, output_type.INT())
       even_sink = DataStreamTestSinkFunction()
       odd_sink = DataStreamTestSinkFunction()
       ds2.add_sink(even_sink)
       ds2.get_side_output(tag).add_sink(odd_sink)

       # multiple_of_three_sink = DataStreamTestSinkFunction()
       # ds2.get_side_output(tag2).add_sink(multiple_of_three_sink) 


       # submit for execution
       env.execute()  

       even_sink.get_result()
       odd_sink.get_result()



   if __name__ == '__main__':
       logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
       parser = argparse.ArgumentParser()
       parser.add_argument('--input',dest='input',required=False,help='Input file to process.')

       argv = sys.argv[1:]
       known_args, _ = parser.parse_known_args(argv)
       side_output(known_args.input)
   ````
 - Iterator Operator
   - IterativeStream을 통해 사용
   - 특정 조건을 만족할때까지 반복(iterate)하며 n번째 iteration의 output이 n+1번째 iteration의 input으로 들어간다 
   - PyFlink는 지원 안하는듯?
     - Java
     ```` java
     public class IterateDemo {
       public static void main(String[] args) throws Exception {
         final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

         // input = [(0,0), (1,0), (2,0), (3,0) (4,0) (5,0)]
         DataStream < Tuple2 < Long, Integer >> data = env.fromSequence(0, 5).map(new MapFunction < Long, Tuple2 < Long, Integer >> () {
             public Tuple2 < Long, Integer > map(Long value) {
               return new Tuple2 < Long, Integer > (value, 0);
             }
           });

         // prepare stream for iteration
         // iterate(max_wait_time) method initiates an iterative part of the program
         IterativeStream < Tuple2 < Long, Integer >> iteration = data.iterate(5000); // ( 0,0   1,0  2,0  3,0   4,0  5,0 )

         // define iteration
         DataStream < Tuple2 < Long, Integer >> plusOne = iteration.map(new MapFunction < Tuple2 < Long, Integer > , Tuple2 < Long, Integer >> () {
             public Tuple2 < Long, Integer > map(Tuple2 < Long, Integer > value) {
               if (value.f0 == 10)
                 return value;
               else
                 return new Tuple2 < Long, Integer > (value.f0 + 1, value.f1 + 1);
             }
           }); //   plusone    1,1   2,1  3,1   4,1   5,1   6,1

         // part of stream to be used in next iteration (
         DataStream < Tuple2 < Long, Integer >> notEqualtoten = plusOne.filter(new FilterFunction < Tuple2 < Long, Integer >> () {
             public boolean filter(Tuple2 < Long, Integer > value) {
               if (value.f0 == 10)
                 return false;
               else
                 return true;
             }
           });
         // feed data back to next iteration
         iteration.closeWith(notEqualtoten);

         // data not feedback to iteration
         DataStream < Tuple2 < Long, Integer >> equaltoten =
           plusOne.filter(new FilterFunction < Tuple2 < Long, Integer >> () {
             public boolean filter(Tuple2 < Long, Integer > value) {
               if (value.f0 == 10) return true;
               else return false;
             }
           });

         equaltoten.writeAsText("/shared/ten");

         env.execute("Iteration Demo");
       }
     }
     ````
 - Flink Optimization in join
   - BROADCAST_HASH_NUM 더 작은 dataset의 hash value들을 node들에게 broadcast
   - REPARTITOIN_HASH_NUM 둘다 클때, 상대적으로 작은 dataset을 쪼개서
   - REPARTITION_SORT_MERGE // 다루지 않음 (data가 커질수록 sorting하는 비용이 너무 커져서)
   ```` java
   DataSet<Tuple3<Integer, String, String>> joined = personSet.join(locationSet, JoinHint.BROADCAST_HASH_FIRST).where(0).equalTo(0)
   DataSet<Tuple3<Integer, String, String>> joined = personSet.join(locationSet, JoinHint.REPARTITION_HASH_FIRST).where(0).equalTo(0)
   ````
   
## Table, SQL API
 - Table API & SQL API → for both stream / batch
   - table 형태로 데이터를 처리하는 컨셉은 같음, 테이블 생성 및 등록에 같은 문법
   - SQL API는 Apache Calcite 기반으로 SQL이랑 거의 같으나, Table API는 그렇지 않음
   - 소싱하여 테이블로 넣는 과정은 크게 2단계
     - 테이블 생성
     - 테이블 등록
   - Table API
   ```` java
   public class TableApiExample {
     public static void main(String[] args) throws Exception {
       EnvironmentSettings settings = EnvironmentSettings
         .newInstance()
         //.inStreamingMode()
         .inBatchMode()
         .build();

       ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
       TableEnvironment tableEnv = TableEnvironment.create(settings);

       final String tableDDL = "CREATE TEMPORARY TABLE CatalogTable (" +
         "date STRING, " +
         "month STRING, " +
         "category STRING, " +
         "product STRING, " +
         "profit INT " +
         ") WITH (" +
         "'connector' = 'filesystem', " +
         "'path' = 'file:///home/jivesh/avg', " +
         "'format' = 'csv'" +
         ")";

       tableEnv.executeSql(tableDDL);

       Table catalog = tableEnv.from("CatalogTable");

       /* querying with Table API */
       Table order20 = catalog
         .filter($("category").isEqual("Category5"))
         .groupBy($("month"))
         .select($("month"), $("profit").sum().as("sum"))
         .orderBy($("sum"));

       // BatchTableEnvironment required to convert Table to Dataset
       BatchTableEnvironment bTableEnv = BatchTableEnvironment.create(env);
       DataSet < Row1 > order20Set = bTableEnv.toDataSet(order20, Row1.class);

       order20Set.writeAsText("/home/jivesh/table1");
       env.execute("State");
     }

     public static class Row1 {
       public String month;
       public Integer sum;

       public Row1() {}

       public String toString() {
         return month + "," + sum;
       }
     }
   }
   ````
   - SQL API
   ```` java
   public class SqlApiExample {
     public static void main(String[] args) throws Exception {

       EnvironmentSettings settings = EnvironmentSettings
         .newInstance()
         //.inStreamingMode()
         .inBatchMode()
         .build();

       ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
       TableEnvironment tableEnv = TableEnvironment.create(settings);

       final String tableDDL = "CREATE TEMPORARY TABLE CatalogTable (" +
         "date STRING, " +
         "month STRING, " +
         "category STRING, " +
         "product STRING, " +
         "profit INT " +
         ") WITH (" +
         "'connector' = 'filesystem', " +
         "'path' = 'file:///home/jivesh/avg', " +
         "'format' = 'csv'" +
         ")";

       tableEnv.executeSql(tableDDL);

       String sql = "SELECT `month`, SUM(profit) AS sum1 FROM CatalogTable WHERE category = 'Category5'" +
         " GROUP BY `month` ORDER BY sum1";
       Table order20 = tableEnv.sqlQuery(sql);

       // BatchTableEnvironment required to convert Table to Dataset
       BatchTableEnvironment bTableEnv = BatchTableEnvironment.create(env);
       DataSet < Row1 > order20Set = bTableEnv.toDataSet(order20, Row1.class);

       order20Set.writeAsText("/home/jivesh/table_sql");
       env.execute("SQL API Example");
     }

     public static class Row1 {
       public String month;
       public Integer sum1;

       public Row1() {}

       public String toString() {
         return month + "," + sum1;
       }
     }
   }
   ````
