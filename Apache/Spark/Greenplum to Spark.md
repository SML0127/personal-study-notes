# Set up druid cluster (feat. hdfs)
Guideline for connecting greenplum to spark

## Reference
 - [https://druid.apache.org/docs/latest/tutorials/cluster.html](https://greenplum-spark.docs.pivotal.io/1-0/using_the_connector.html)
 - https://github.com/kongyew/greenplum-spark-connector

## Set up guideline
 - Download greenplum-spark connector
   - 링크: [Download VMware Tanzu™ Greenplum® — VMware Tanzu Network (pivotal.io)](https://network.pivotal.io/products/vmware-tanzu-greenplum#/releases/280281/file_groups/702)

 - greenplum-connector-apache-spark-scala_2.12-2.1.1.jar 등록
   - spark-shell에서 테스트? GSC_JAR 환경 변수에 greenplum-connector-apache-spark-scala_2.12-2.1.1.jar 경로 등록
   - zeppelin에서 테스트?  SPARK_SUBMIT_OPTIONS에 hdfs://hadoop_cluster/your_path/greenplum-connector-apache-spark-scala_2.12-2.1.1.jar 추가


 - Start spark-shell with connector
   - spark-shell --jars $GSC_JAR


 - 설치 확인 명령어
   - Class.forName("io.pivotal.greenplum.spark.GreenplumRelationProvider")
   - Class.forName("org.postgresql.Driver")
     ![image](https://user-images.githubusercontent.com/13589283/171171371-83685600-db18-4974-8efd-96dbce2d50d2.png)

