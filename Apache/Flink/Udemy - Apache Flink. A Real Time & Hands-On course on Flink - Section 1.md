# Section 1: Introduction Apache Flink | A Real Time & Hands-On course on Flink Section 1

## Reference
https://www.udemy.com/course/apache-flink-a-real-time-hands-on-course-on-flink/


## Contents
### Flink?
 - open-source stream processing framework for distributed, high-performing, always available and data streaming applications
 - 3가지 유형의 연산 지원 (Flink가 제공하는 대부분의 기능은 spark도 지원)
   - Batch processing
   - Graph processing
   - Iterative processing (for ml)


 - 그럼 Spark를 쓰면되지 않느냐? Spark보다 조금 더 빨라서 (Flink > Spark > Hadoop)


 - 그외 특징
   - Low latency (stream processing을 잘 지원 한다는 의미)
   - High throughput app
   - Robust fault-tolerance
   - application이 실패한 그 정확한(exactly) 시점(point)에서 restart가 가능
   - Rich set of libraries for Graph processing, ML, String handling, relational APIs
   - re-scalable (in runtime) == App 실행 중에도 resource 증가 지원
   - Maintaining exactly one semantics == 모든 레코드는 단 한번만 처리되는 것을 guarantee


 - Flink는 Batch와 Streaming processing 둘다 지원
   - Batch?
     - 일정 주기 또는 on-demand 형태로 bounded dataset을 처리
     - throughput(단위 시간당 데이터 처리량)에 초점
<img width="464" alt="image2022-6-15_17-48-16" src="https://user-images.githubusercontent.com/13589283/174443985-fc99d210-b8ce-4253-a056-cb2bfefce42f.png">


   - Streaming?
     - rea-time data를 처리하는데 사용 (e.g. Fraud detection, social media sentiment analysis)
     - unbounded data를 끊김 없이 처리(continuously whenever data is available) 
     - latency(일정량의 데이터를 처리하는데 걸리는 시간) 초점


 - Hadoop vs Spark, Flink
   - Hadoop은 모든 mapper, reducer의 결과물인 intermediate result를 disk에 write하지만,
   - Spark와 Flink는 최초 read, 마지막 write 제외하고 메모리에 유지 


 - Spark:RDD ==  Flink:Dataflows


 - ML을 위해 Hadoop은 별도의 tool(e.e apache mahout)이 필요하나 Spark나 Flink는 Mlib, FlinkML 각각의 library가 존재 및 제공



 - Spark가 stream processing을 지원하지만, 초기 RDD의 batch processing을 기반으로 고안(not a true real time processing)


 - Spark streaming computation model은 micro-batching 기반, batch가 작을 수록 near real time processing으로 유지


 - Flink는 window 개념이 존재
   - data는 window에 쌓이고, 일정 시간이 지나면 flink engine(실제 연산을 수행하는 프로세스)으로 전송
   - window마다 checkpoint를 두어 실패시 restart 지원
 
 - Flink는 자체적으로 효율적인 automatic memory manager 가지고 있어서 memory management관점에서 spark 보다 우위(upper hand)에 있다
   - virtual memory로 스왑 지원


 - Spark가 DAG라면, Flink는 cyclic dependency graph 형태로 제어 (in runtime)
   - cycle이 존재 할 수 있어서 ML 알고리즘시 DAG에 비해 더 효율적으로 처리 할 수 있다는듯?


 - Batch workload (e.g. word count, tera sora)와 Iterative workload (e.g. page rank, k-means)에서 대부분 Flink가 좀 더 빠름 (모든 실험에 대해서는 아니고..)
 
 - Apache spark에 비해서는 현저히 낮은 oom 에러 리포트...


 - Flink architecture
   - Storage
     - File: local, HDFS, S3
     - DB: MongoDB, HBase
     - Sreams: Kafka, Flume, RabbitMq


   - Deploy
     - Local JVM
     - Cluster (Standalone, yarn)
     - Cloud (GCE, EC2)


   - Engine == FLINK's RUNTIME

   - Abstraction
     - DataSet (Batch)
       - TABLE for Relation 처리
       - GELLY for Graph 처리
       - FLINK ML for ML
     - DataStream (Stream)

   - {DataSet, DataStream} > Flinks' RUNTIME (engine)



 - programming model (flow of a flink program)
   - Source → Operations / Transformation → Sink
     - Source: file, kafka, flume, socket
     - Sink: HDFS, DB, Memory


   - Example
     - File을 읽고 처리한다고 생각해보자
     - File은 여러개의 block (이게 window 개념인가?)으로 나누어지고 또 여러 node에서 처리된다.
     - node에 문제가 발생하면 flink는 새로운 node(new active node)를 할당하여 처리하지 못한 block을 처리하게 한다.


   - Flink는 매 연산의 결과로 dataset or datastream을 생성
     - Spark에서 transform/action의 결과로 RDD를 새로 만드는 거랑 똑같은데?
     - spark 연산의 근간이 RDD 이듯 flink 연산의 근간은 DataSet/DataFrame이다
     - 연산 이후 새로운 dataset / datastream을 생성하는게 아니고 연산 중에 만든다?
       - java에서는 DataSet/DataStream 명시해줘야하고, scala는 자동으로 설정해줌 


 - DataSet / DataStream
   - immutable (like RDD)
   - only one operator for one dataset/dataframe (cannot perform different operations to same dataset/dataframe) → 표현이 애매한데, dataset의 일부에 A 연산 나머지 일부에 B 연산 이런게 안된다는 뜻이고 dataset 전체에 대해 A 연산, B 연산 각각 수행은 가능
   - dependency가 있음 (연산의 순서) → 각 datatset들은 list of dependency를 가지고 있고, 수행해야할 모든 연산 tracking함
