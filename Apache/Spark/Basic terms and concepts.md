## Spark란?
Disk-based인 하둡을 in-memory로 변환하여 처리 성능을 높인 분산 처리 프레임워크     

## Spark & Spark cluster에서 사용되는 개념
 - SparkContext
   - Spark Application 관련 정보(e.g. app name, configuration)등을 가지고 있는 entry point
   - Resource Manager를 이용하여 Spark Application이 Spark cluster로 연결되는것 가능케 함
   - Context object를 생성하여 RDD 생성 및 각종 연산에 사용 
   - Context 마다 별도의 object를 생성해야하는 단점 존재
     - SQLContext, HiveContext, StreamingContext


 - SparkSession (Spark ver >= 2.0)
   - SparkContext를 포함, context 마다 context object를 생성할 필요성을 제거
   - SparkContext 특징들을 포함하여 DataSet, DataFrame API의 entry point 역할
    ```` scala
    import org.apache.spark.sql.SparkSession
 
    val spark = SparkSession
      .builder()
      .appName("Spark Session Example")
      .config("spark.some.config.option", "some-value")
      .getOrCreate()
    val df_ㅜ ㅁfor_parquet = spark.read.parquet("file.parquet")
    val df_for_json = spark.read.parquet("file.json")
    ````


 - Spark Application
   - main 함수를 실행하는 driver와, driver가 할당한 task를 수행하는 executor들로 구성, executor는 cluster manager에게 할당 받음
   - driver는 main함수와 parallel operation을 수행, parallel operation은 데이터를 읽어들여 RDD, dataset, dataframe으로 만드는 작업
   - executor는 SparkContext로 부터 할당 받은 task(application code == jar, data)들을 처리, 데이터(연산의 중간/최종 결과물) driver로 전송


 - Cluster Manager (Resource Manager)
   - cluster 모드로 spark를 수행시, Spark application은 context object를 통해 cluster manager에 연결되며, cluster manager는 application에게 resource를 할당 

   - local, standalone, yarn?
     - standalone, yarn = cluster, local = single machine
     - local[N]은 driver, executor를 포함한 모든 것들이 하나의 머신에서 수행
     - Standalone은 cluster 위의 별개의 jvm위에서 각각 master와 worker들이 동작, 1 executor per worker node 
     - yarn은 하둡 기반의 리소스 매니저로, 원하는 데이터가 저장된 hdfs 위에서 spark application이 수행
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![0_XwWZAbmHF75xWRDR](https://user-images.githubusercontent.com/13589283/150999597-6b22dc10-29ee-4b44-b7b1-129167198460.png)

 - 배포 방식? Client? Cluster? 
   - driver가 어디서 실행되는지의 차이. cluster 외부 머신에 있으면 client, cluster 내부 worker에 있으면 cluster
   - client는 spark-submit을 실행하는 머신에서 real-time으로 입/출력이 가능, real-time 입/출력이 가능하여 개발과정에서 대화형 디버깅 가능
   - cluster mode면 spark-submit으로 application 제출하고 submit에 사용한 머신을 꺼도 돌아감



## Spark 관련 데이터 타입
 - RDD
   - Parallel하게 처리 될 수 있도록 클러스터의 노드들에게 파티셔닝되는 element들의 collection (a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel)
   - HDFS, File등 외부 데이터들을 context에서 제공되는 api를 이용하여 읽을 경우, 내부적으로 parallelize되며 그 외에 코드상에 데이터를 입력하여 쓸 경우 명시적으로 parallelize가 필요
    ```` scala
    // Array to RDD
    val data = Array(1, 2, 3, 4, 5)
    val distData = sc.parallelize(data)

    // File to RDD
    val distFile = sc.textFile("data.txt")
    ````



 - DataFrame
   - Spark ver 1.3에서 등장, 개념적으로 RDB의 table과 python의 dataframe과 동일
   - file, table (in hive or db), RDD들로 DataFrame 생성 가능
   - Spark ver > 2.0에서는, 파일을 읽거나 sql의 결과등의 결과물은 대부분 DataFrame이다라 생각해도 될 듯
   ```` scala
   // RDD to DF
   case class sample_class(col1_name: Int, col2_name: Int)
   val df = rdd.map(x -> sample_class(x(0), x(1))).toDF()

   // RDD to DF with column name
   val df = rdd.toDF("col1_name", "col2_name")

   // Result type is DataFrame
   val df1 = spark.sql("select * from table")
   val df2 = spark.read.json("path/to/file.json")
   ````

 - DataSet
   - Spark er 1.6에서 등장, 2.0으로 넘어오면서 RDD는 Dataset으로 대체(replace)되고 DataFrame과 통합
   - Scala에서 DataSet[row]는 DataFrame과 같음 (alias 관계)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![31july_rdd_df_ds](https://user-images.githubusercontent.com/13589283/172053979-f34ba1b4-5330-4fd1-91bc-9a79a40b0d60.png)

   - RDD와 매우 유사한 타입(strongly-typed like an RDD)이며 SparkSQL 최적화된 execution engine의 이점을 가지고 있음
     - 즉, 기존 RDD의 transform과 action, SparkSQL의 api들 둘다 사용가능
   ```` scala
   // RDD to DS
   case class class_name(col1_name: Int, col2_name: Int)
   val caseClassDS = Seq(class_name(value1, value2)).toDS()

   // Encoders for most common types are automatically provided by importing spark.implicits._
   val primitiveDS = Seq(1, 2, 3).toDS()
   ````

## Spark cluster에서 Spark application이 동작되는 과정
 1. spark-submit을 통해 spark application(jar)이 제출
 2. 배포방식(client, cluster)과 cluster manager에 따라 차이가 있으나, driver가 main 함수를 수행하며 spark context를 생성
 3. spark context를 통해 cluster manager와 연결되고, cluster manager는 spark application에게 executor들 할당
 4. driver는 executor들에게 task(code, data)들을 할당
 
 
 
 ### Reference
  - https://databricks.com/kr/blog/2016/08/15/how-to-use-sparksession-in-apache-spark-2-0.html
  - https://databricks.com/kr/glossary/what-are-spark-applications
  - https://www.ksolves.com/blog/big-data/spark/sparksession-vs-sparkcontext-what-are-the-differences
  - https://spark.apache.org/docs/2.3.0/sql-programming-guide.html
  - https://spark.apache.org/docs/2.3.0/sql-programming-guide.html
 
