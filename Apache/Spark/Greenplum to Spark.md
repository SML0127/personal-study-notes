# Set up to connect greenplum with spark
Guideline for connecting greenplum to spark

## Reference
 - [https://druid.apache.org/docs/latest/tutorials/cluster.html](https://greenplum-spark.docs.pivotal.io/1-0/using_the_connector.html)
 - https://github.com/kongyew/greenplum-spark-connector

## Set up guideline
 - Download greenplum-spark connector
   - link: [Download VMware Tanzu™ Greenplum® — VMware Tanzu Network (pivotal.io)](https://network.pivotal.io/products/vmware-tanzu-greenplum#/releases/280281/file_groups/702)

 - Add greenplum-connector-apache-spark-scala_2.12-2.1.1.jar
   - spark-shell → set environment variable GSC_JAR = path to greenplum-connector-apache-spark-scala_2.12-2.1.1.jar
   - zeppelin →  add hdfs://hadoop_cluster/your_path/greenplum-connector-apache-spark-scala_2.12-2.1.1.jar to SPARK_SUBMIT_OPTIONS


 - Start spark-shell with connector
   ```` script
   spark-shell --jars $GSC_JAR
   ````


 - Installation result
   ```` scala
   Class.forName("io.pivotal.greenplum.spark.GreenplumRelationProvider")
   Class.forName("org.postgresql.Driver")
   ````
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/13589283/171171371-83685600-db18-4974-8efd-96dbce2d50d2.png)

## Toy Example

 - Connect to Greenplum 
   ```` sql
   psql -h server_ip -p server_port -U user_name -d database_name
   ````
 - Change user role
   ```` sql
   alter role user_name with CREATEEXTTABLE (type='writable',protocol='gpfdist');
   ````

 - Create table and insert dumy data
   ```` sql
   CREATE TABLE test ( col1 int, col2 int, col3 text ) DISTRIBUTED BY (col1, col2);
   insert into test_table(col1, col2, col3) values(1, 1, 'number 1, 1');
   insert into test_table(col1, col2, col3) values(1, 2, 'number 1, 2');
   insert into test_table(col1, col2, col3) values(1, 3, 'number 1, 3');
   insert into test_table(col1, col2, col3) values(1, 4, 'number 1, 4');
   insert into test_table(col1, col2, col3) values(1, 5, 'number 1, 5');
   insert into test_table(col1, col2, col3) values(2, 1, 'number 2, 1');
   insert into test_table(col1, col2, col3) values(2, 2, 'number 2, 2');
   insert into test_table(col1, col2, col3) values(2, 3, 'number 2, 3');
   insert into test_table(col1, col2, col3) values(2, 4, 'number 2, 4');
   insert into test_table(col1, col2, col3) values(2, 5, 'number 2, 5');
   ````
 - Test command
   ```` scala
   val df = spark.read.format("greenplum").option("url", "jdbc:postgresql://server_ip:server_port/database").option("user", "user_name").option("password", "your_password").option("dbschema","schema_name").option("dbtable", "test_table").option("partitionColumn","col1").option("partitionColumn","col2").load()
   df.createTempView("test_table")
   spark.sql("select * from test_table").show()
   ````
 - Result <br><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/13589283/171176088-f79b3546-b69a-4700-a162-651dd9ee0b66.png)



   
