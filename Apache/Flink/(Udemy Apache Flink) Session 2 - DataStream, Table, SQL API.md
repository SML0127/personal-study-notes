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
