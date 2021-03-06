# Set up druid cluster (feat. hdfs)
Guideline for setting up druid cluster and load data from hdfs

## Reference
 - https://druid.apache.org/docs/latest/tutorials/cluster.html

## Requirements
 - Java >= 1.8 
(Druid officially supports Java 8 only. Support for later major versions of Java is currently in experimental status.)

## Set up guideline
### 1. Download and unzip druid 
 - Download druid
   - wget https://downloads.apache.org/druid/0.22.1/apache-druid-0.22.1-bin.tar.gz

 - Unzip
   - tar -xzf apache-druid-0.22.1-bin.tar.gz
  
### 2. Configure runtime.properties in Master / Data / Query server
Configure it on one server and then copy it to other servers (step 7)

 - Master server 
   ```` xml
   druid.host = master_server_ip (in coordinator-overload/runtime.properties)
   ````


 - Data server
   ```` xml
   druid.host=historical_server_ip (in historical/runtime.properties)
   druid.host=middlemanager_server_ip (in middleManager/runtime.properties)
   druid.host=indexer_server_ip (in indexer/runtime.properties)
   ````

 - Query server
   ```` xml
   druid.host=broker_server_ip (in broker/runtime.properties)
   druid.host=router_server_ip (in router/runtime.properties)
   ````

### 3. Configure master (in conf/druid/cluster/_common/common.runtime.properties)
 - host 
   ```` xml
   druid.host=localhost
   ````

 - Metadata storage 
   ```` xml
   druid.metadata.storage.type=derby
   druid.metadata.storage.connector.connectURI=jdbc:derby://master_server_ip:1527/var/druid/metadata.db;create=true
   druid.metadata.storage.connector.host=master_server_ip
   druid.metadata.storage.connector.port=1527
   ````

 - Deep storage 
   ```` xml
   druid.extensions.loadList에 druid-hdfs-storage 추가
   druid.storage.type=hdfs
   druid.storage.storageDirectory=/your_hdfs_path/druid/segments
   druid.indexer.logs.type=hdfs
   druid.indexer.logs.directory=/your_hdfs_path/druid/indexing-logs
   ````

### 4. Configure connection to Hadoop 

 - Copy hadoop configuration XMLs(core-site.xml, hdfs-site,xml, yarn-site.xml, mapred-site.xml) to conf/druid/cluster/_common/
 - (Optional) Copy krb5.conf for Kerberos


### 5. Configure Zookeeper 
```` xml
druid.zk.service.host=zookeeper_server_ip:port
druid.zk.paths.base=/druid
````

### 6. (Optional) Configure memeory size
Configure base on your server spec

 - Data node 
   - Historical server 
     - jvm (in apache-druid-0.22.1/conf/druid/cluster/data/historical/jvm.config)
       ```` xml  
       Xms256m  # Set using size of server swap memeory  
       Xmx256m 
       XX:MaxDirectMemorySize=3g  # MaxDirectMemorySize >= sizeBytes * (numMergeBuffers + numThreads + 1)
       ````
     - runtime properties (in apache-druid-0.22.1/conf/druid/cluster/data/historical/runtime.properties)
       ```` xml
       druid.processing.buffer.sizeBytes=250MiB  # Keep this unchanged
       druid.processing.numMergeBuffers=4  # Divide the old value from the single-server deployment by the split factor
       druid.processing.numThreads=7  # Set to (num_cores - 1) based on the new hardware
       ````

   - Middle manager
     - jvm (in apache-druid-0.22.1/conf/druid/cluster/data/middleManager/jvm.config)
       ```` xml
       Xms128m
       Xmx128m 
       ````

     - runtime properties (in apache-druid-0.22.1/conf/druid/cluster/data/middleManager/runtime.properties) 
       ```` xml
       druid.worker.capacity=8  # default는 8 그러면 2대 기준 4? Divide the old value from the single-server deployment by the split factor
       druid.indexer.runner.javaOpts=-server -Xms128m -Xmx128m -XX:MaxDirectMemorySize=3g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+E    xitOnOutOfMemoryError -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
       ````


 - Query node
   - Broker 
     - jvm (in apache-druid-0.22.1/conf/druid/cluster/query/broker/jvm.config)
       ```` xml
       Xms128m
       Xmx128m
       XX:MaxDirectMemorySize=3g  # MaxDirectMemorySize >= sizeBytes * (numMergeBuffers + numThreads + 1)
       ````

     - runtime properties (in apache-druid-0.22.1/conf/druid/cluster/query/broker/runtime.properties)
       ```` xml
       druid.processing.buffer.sizeBytes=250MiB
       druid.processing.numMergeBuffers=4
       druid.processing.numThreads=1
       ````

   - Router 
     - jvm (in apache-druid-0.22.1/conf/druid/cluster/query/router/jvm.config)
       ```` xml
       Xms256m
       Xmx256m
       XX:+UseG1GC
       XX:MaxDirectMemorySize=128m
       ````

### 7. Copy configuration
Copy all configurations to other servers 


### 8. Start server
 - Master
   - bin/start-cluster-master-with-zk-server

 - Data server 
   - bin/start-cluster-data-server
 
 - Query server
   - bin/start-cluster-query-server



### 9. Check admin page  
 - Master: http://master_server_ip:8081
 - Querynode: http://query_server_ip:8888/

### 10. Test using sample data wikiticker
 - Load data  
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/13589283/170643679-0068dfb0-5d6c-4bff-8fbd-e683cac9986d.png">

 - Ingestion
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/13589283/170643718-bb0f3cc6-2c39-4ca4-8071-e35a7f823683.png">

 - Check datasource
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/13589283/170643732-5c49352b-bbcc-43d3-b059-31da00562bac.png">

 - Querying
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/13589283/170643751-98fc4fe8-e052-42c7-b592-5fbaf6848852.png">

