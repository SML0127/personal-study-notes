# Section 3: Windows, Trigger, Evictor

## Windows
 - Window in stream? → datastream을 bucket으로 나눈 것 (Windows split the data stream into buckets of finite size over which computations can be applied)

 - Window 생성 조건
   - Time
     - Tumbling window ,Sliding window (support overlap) → sliding window is better network bandwidth, reliable delivery of packet
     - Key가 있는 stream(window())과 key가 없는 stream(windowAll()) 으로 분류하여 지원
       - windowAssigner (어떻게 entity를 window에 할당할지)를 인자로 받음 

   - Specific event
   - the number of entity(data, record, element)

 - Time의 분류
   - Processing time
     - System이 task를 수행하는데 걸리는 시간
     - Window는 machine의 system clock을 사용
     - Best performance and low latency
     - Less suitable for distributed env
     - 매 operation 마다 window에 새로운 시간을 할당하기에 변함 (이것 때문에 event / ingestion time이 stable하다고 말하기도함) 

   - Event time
     - Source에서 이벤트가 발생한 시간 (Source → Flink Ingestion → Processin)
     - record안에 포함됨(embedded)
     - 순서에 상관없이 deterministic한 result를 도출
     - Out-of-order event를 기다릴 경우 latency가 발생

   - Ingestion time
     - 개념적으로 event time과 processing time 사이
     - event time과 거의 비슷하게 다뤄지나, out-of-order event / late data를 처리 못하는 차이 존재
<img width="605" alt="image2022-6-17_22-49-59" src="https://user-images.githubusercontent.com/13589283/178994358-ae2f4d3b-03fe-43c1-bb88-76ba3bf9a9e7.png">


 - Tumbling window 예제
![tumbling-windows](https://user-images.githubusercontent.com/13589283/178994656-ef4ecb9f-a28a-430c-8a8f-1155843cb5b4.svg)

   - Java
```` java
public class AverageProfitTumblingProcessing {
  public static void main(String[] args) throws Exception {
    // set up the streaming execution environment
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    //env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
 
    DataStream < String > data = env.socketTextStream("localhost", 9090);
 
    // month, product, category, profit, count
    DataStream < Tuple5 < String, String, String, Integer, Integer >> mapped = data.map(new Splitter()); // tuple  [June,Category5,Bat,12,1]                                                                                       
    DataStream < Tuple5 < String, String, String, Integer, Integer >> reduced = mapped
      .keyBy(t -> t.f0)
      .window(TumblingProcessingTimeWindows.of(Time.seconds(2)))
    //.window(SlidingProcessingTimewindows.of(Time.seconds(2), Time.seconds(1))
      .reduce(new Reduce1());
    // June { [Category5,Bat,12,1] Category4,Perfume,10,1}  //rolling reduce           
    // reduced = { [Category4,Perfume,22,2] ..... }
    reduced.addSink(StreamingFileSink
      .forRowFormat(new Path("/home/jivesh/www"),
        new SimpleStringEncoder < Tuple5 < String, String, String, Integer, Integer >> ("UTF-8"))
      .withRollingPolicy(DefaultRollingPolicy.builder().build())
      .build());
 
    // execute program
    env.execute("Avg Profit Per Month");
  }
 
  public static class Reduce1 implements ReduceFunction < Tuple5 < String, String, String, Integer, Integer >> {
    public Tuple5 < String, String, String, Integer, Integer > reduce(Tuple5 < String, String, String, Integer, Integer > current,
      Tuple5 < String, String, String, Integer, Integer > pre_result) {
      return new Tuple5 < String, String, String, Integer, Integer > (current.f0,
        current.f1, current.f2, current.f3 + pre_result.f3, current.f4 + pre_result.f4);
    }
  }
  public static class Splitter implements MapFunction < String, Tuple5 < String, String, String, Integer, Integer >> {
    public Tuple5 < String, String, String, Integer, Integer > map(String value) // 01-06-2018,June,Category5,Bat,12
    {
      String[] words = value.split(","); // words = [{01-06-2018},{June},{Category5},{Bat}.{12}
      // ignore timestamp, we don't need it for any calculations
      //Long timestamp = Long.parseLong(words[5]);
      return new Tuple5 < String, String, String, Integer, Integer > (words[1], words[2], words[3], Integer.parseInt(words[4]), 1);
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
from pyflink.common.watermark_strategy import WatermarkStrategy, TimestampAssigner
from pyflink.datastream.window import (TumblingEventTimeWindows, TumblingProcessingTimeWindows, SlidingEventTimeWindows, EventTimeSessionWindows, CountSlidingWindowAssigner, SessionWindowTimeGapExtractor, CountWindow, PurgingTrigger, EventTimeTrigger, TimeWindow, GlobalWindows, CountTrigger)
 
 
def tumbling_window_avg_profit(input_path):
    env = StreamExecutionEnvironment.get_execution_environment()
    env.set_runtime_mode(RuntimeExecutionMode.BATCH)
    env.set_parallelism(1)
 
    # define the source
    if input_path is not None:
        data_stream = env.from_source(
            source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
            # watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
            source_name="file_source"
        )
    else:
        print("Executing word_count example with default input data set.")
        print("Use --input to specify file input.")
        return
         
 
    watermark_strategy = WatermarkStrategy.for_monotonous_timestamps() \
                                              .with_timestamp_assigner(SecondColumnTimestampAssigner())
    # month, product, category, profit, count
    res_stream = DataStreamTestSinkFunction()
    data_stream.assign_timestamps_and_watermarks(watermark_strategy) \
               .map(lambda i: (i[0], i[1], i[2], float(i[4]), 1)) \
               .key_by(lambda x: x[0], key_type=Types.STRING()) \
               .window(TumblingProcessingTimeWindows.of(Time.milliseconds(2000))) \
               .reduce(lambda i, j: (i[0], i[3] + j[3], i[4] + j[4])) \
               .map(lambda i: (i[0], float(i[3] / i[4])) ) \ 
               .add_sink(res_stream)
 
    # submit for execution
    env.execute()  
 
    print(res_stream.get_results())
 
if __name__ == '__main__':
    logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
    parser = argparse.ArgumentParser()
    parser.add_argument('--input',dest='input',required=False,help='Input file to process.')
 
    argv = sys.argv[1:]
    known_args, _ = parser.parse_known_args(argv)
    tumbling_window_avg_profit(known_args.input)
````


 - Sliding windows 예제
   - Example
     - Python
     ```` python
     data_stream.key_by(lambda i: i[0])
           .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5))) \
     ````

 - Session windows
   - created based on activity
   - does not have fixed start or end time → session 개념과 동일, active ~ inactive 하는 동안에 들어오는 데이터들이 하나의 window 가 되는것. 따라서 고정된 시작 / 끝 시간은 없음
   - gap 이라는 시간 내에 data가 들어오지 않으면 그 앞까지의 data들을 하나의 window가 된다. → data가 들어오면 active가 되고, 지정한 gap안에 안들어오면 inactive상태가 된다.
   - Example
     - Java
       ```` java
       public class AverageProfitSessionWindowProcessingTime {
         public static void main(String[] args) throws Exception {
           StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
 
           DataStream < String > data = env.socketTextStream("localhost", 9090);
 
           // month, product, category, profit, count
           DataStream < Tuple5 < String, String, String, Integer, Integer >> mapped = data.map(new Splitter()); // tuple  [June,Category5,Bat,12,1]                                                                                       
           DataStream < Tuple5 < String, String, String, Integer, Integer >> reduced = mapped
             .keyBy(t -> t.f0)
             .window(ProcessingTimeSessionWindows.withGap(Time.seconds(1)))
             .reduce(new Reduce1());
 
           reduced.addSink(StreamingFileSink
             .forRowFormat(new Path("/home/jivesh/www"),
               new SimpleStringEncoder < Tuple5 < String, String, String, Integer, Integer >> ("UTF-8"))
             .withRollingPolicy(DefaultRollingPolicy.builder().build())
             .build());
 
           // execute program
           env.execute("Avg Profit Per Month");
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
       from pyflink.common.watermark_strategy import WatermarkStrategy, TimestampAssigner
       from pyflink.datastream.window import (TumblingEventTimeWindows, TumblingProcessingTimeWindows, ProcessingTimeSessionWindows, SlidingEventTimeWindows, EventTimeSessionWindows, CountSlidingWindowAssigner, SessionWindowTimeGapExtractor, CountWindow, PurgingTrigger, EventTimeTrigger, TimeWindow, GlobalWindows, CountTrigger)
 
 
       def session_window_avg_profit(input_path):
           env = StreamExecutionEnvironment.get_execution_environment()
           env.set_runtime_mode(RuntimeExecutionMode.BATCH)
           env.set_parallelism(1)
 
           # define the source
           if input_path is not None:
               data_stream = env.from_source(
                   source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
                   # watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
                   source_name="file_source"
               )
           else:
               print("Executing word_count example with default input data set.")
               print("Use --input to specify file input.")
               return
         
 
           watermark_strategy = WatermarkStrategy.for_monotonous_timestamps() \
                                              .with_timestamp_assigner(SecondColumnTimestampAssigner())
           # month, product, category, profit, count
           res_stream = DataStreamTestSinkFunction()
           data_stream.assign_timestamps_and_watermarks(watermark_strategy) \
                      .map(lambda i: (i[0], i[1], i[2], float(i[4]), 1)) \
                      .key_by(lambda x: x[0], key_type=Types.STRING()) \
                      .window(ProcessingTimeSessionWindows.with_gap(Time.milliseconds(1000))) \
                      .reduce(lambda i, j: (i[0], i[3] + j[3], i[4] + j[4])) \
                      .map(lambda i: (i[0], float(i[3] / i[4])) ) \ 
                      .add_sink(res_stream)
 
           # submit for execution
           env.execute()  
 
           print(res_stream.get_results())
 
       if __name__ == '__main__':
           logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
           parser = argparse.ArgumentParser()
           parser.add_argument('--input',dest='input',required=False,help='Input file to process.')
 
           argv = sys.argv[1:]
           known_args, _ = parser.parse_known_args(argv)
           session_window_avg_profit(known_args.input)
       ````
 ![session-windows](https://user-images.githubusercontent.com/13589283/178997080-2fb40617-60bc-4abc-91b7-dc4a86b0a71e.svg)
      
 - Global window
   - one window per key → key 별로 하나의 window만 유지, processing 되기 위해 trigger 필요
   - # of data, time 같은 trigger 사용
   - Example
     - Java
     ```` java
     DataStream < Tuple5 < String, String, String, Integer, Integer >> reduced = mapped
     .keyBy(t -> t.f0)
     .window(GlobalWindows.create()))
     .trigger(CounTrigger.of(5))
     .reduce(new Reduce1());
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
     from pyflink.common.watermark_strategy import WatermarkStrategy, TimestampAssigner
     from pyflink.datastream.window import (TumblingEventTimeWindows, TumblingProcessingTimeWindows, ProcessingTimeSessionWindows, SlidingEventTimeWindows, GlobalWindows EventTimeSessionWindows, CountSlidingWindowAssigner, SessionWindowTimeGapExtractor, CountWindow, PurgingTrigger, EventTimeTrigger, TimeWindow, GlobalWindows, CountTrigger)
 
 
     def global_window_avg_profit(input_path):
         env = StreamExecutionEnvironment.get_execution_environment()
         env.set_runtime_mode(RuntimeExecutionMode.BATCH)
         env.set_parallelism(1)
 
         # define the source
         if input_path is not None:
             data_stream = env.from_source(
                 source=FileSource.for_record_stream_format(StreamFormat.text_line_format(), input_path).process_static_file_set().build(),
                 # watermark_strategy=WatermarkStrategy.for_monotonous_timestamps(),
                 source_name="file_source"
             )
         else:
             print("Executing word_count example with default input data set.")
             print("Use --input to specify file input.")
             return
         
 
         watermark_strategy = WatermarkStrategy.for_monotonous_timestamps() \
                                                   .with_timestamp_assigner(SecondColumnTimestampAssigner())
         # month, product, category, profit, count
         res_stream = DataStreamTestSinkFunction()
         data_stream.assign_timestamps_and_watermarks(watermark_strategy) \
                    .map(lambda i: (i[0], i[1], i[2], float(i[4]), 1)) \
                    .key_by(lambda x: x[0], key_type=Types.STRING()) \
                    .window(GlobalWindows.create()) \
                    .trigger(CountTrigger(5))
                    .reduce(lambda i, j: (i[0], i[3] + j[3], i[4] + j[4])) \
                    .map(lambda i: (i[0], float(i[3] / i[4])) ) \ 
                    .add_sink(res_stream)
 
         # submit for execution
         env.execute()  
 
         print(res_stream.get_results())
 
     if __name__ == '__main__':
         logging.basicConfig(stream=sys.stdout, level=logging.INFO, format="%(message)s")
         parser = argparse.ArgumentParser()
         parser.add_argument('--input',dest='input',required=False,help='Input file to process.')
 
         argv = sys.argv[1:]
         known_args, _ = parser.parse_known_args(argv)
         global_window_avg_profit(known_args.input)
     ````

## Trigger
 - Trigger 
   - 언제 윈도우가 처리될 준비 완료 되었는지 결정(determines when a window is ready to be processed)
   - All window assigners comes with default triggers
   - 5 methods in trigger interface 
     - TriggerResult onElement() → element가 window로 들어올때마다 호출. window에 들어오는 element의 수 카운트하고 싶을때 사용
     - TriggerResult onEventTime() → window에서 특정 시간이 경과할때마다 호출
     - TriggerResult onProcessingTime() → processing단에서 특정 시간이 경과댈때마다 호출
     - void onMerge() → merge 될때마다 호출 (e.g. session window)
     - void clear() → clear window

   - 4 TriggerResult (can add custom context)
     - CONTINUE: Do nothing
     - FIRE: Trigger the computation
     - PURGE: Clear contents of windows
     - FIRE_AND_PURGE: first fire and then purge

   - Example of pre-defined triggers (overridden 5 methods)
      - Java
      ```` java
      // fire based on progress of event time
      .trigger(EventTimeTrigger.create())
 
      // fire based on progress of processing time
      .trigger(ProcessingTimeTrigger.create())
 
      // fire based when the num of element exceeds
      .trigger(CountTrigger.of(max_num_of_element))
 
      // takes another trigger and purge it after the inner one fires
      .trigger(PurgingTrigger.of(CountTrigger.of(max_num_of_element)))
      ````
      - Python
      ```` python
      from pyflink.datastream.window import (EventTimeTrigger, ProcessingTimeTrigger, CountTrigger, PurgingTrigger)
 
      // fire based on progress of event time
      .trigger(EventTimeTrigger.create())
 
      // fire based on progress of processing time
      .trigger(ProcessingTimeTrigger.create())
 
      // fire based when the num of element exceeds
      .trigger(CountTrigger(max_num_of_element))
 
      // takes another trigger and purge it after the inner one fires
      .trigger(PurgingTrigger.of(CountTrigger(max_num_of_element)))
      ````

## Evictor
 - Evictor
   - Window가 trigger된 후, window function 적용 전 또는 후에 window에 있는 data를 삭제하는 기능 (used to remove elements from a window after the trigger fires and before and/or after the window function is applied)
   - PyFlink에서는 현재 지원x
   - Window의 lifecycle
     - window created → trigger → window function(e.g. reduce) → result
     ```` java
     data_stream.window(GlobalWindows.create()) # window created (첫 element가 들어왔을때 생성)
           .trigger(CountTrigger(5)) # trigger window
           .reduce(lambda i, j: (i[0], i[3] + j[3], i[4] + j[4])) # window function 
           .add_sink(res_stream) # result
     ````

   - trigger / window funcion 사이(evictBefore()) 또는 window function → result 사이(evictAfter())에 evictors 는 들어갈 수 있다. (동시도 가능)
   - 개수, 시간, element의 어떤 특성에 따라 evict 가능
   - custom 가능
   - Example of 3 pre-defined built-in evitors
   ```` java
   
    // max_num_of_element 개수 까지만 element 유지
    .evictor(CountEvictor.of(max_num_of_element))

    // last element와 window에 있는 element들간의 delta를 계산하여 threshold보다 작으면 window에 유지
    .evictor(DeltaEvictor.of(threshold, new MyDelta()))

    // window내의 element중 가장 큰 timestamp를 max_timestamp로 하여, timestamp가 (max_timestamp - evictionSec)보다 작은 element들을 discard 
    .evictor(TimeEvictor.of(Time.of(evictionSec, TimeUnit.SECONDS)))
   ````
