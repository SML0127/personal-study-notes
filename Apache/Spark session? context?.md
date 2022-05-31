## Spark란?
Disk-based인 하둡을 in-memory로 변환하여 처리 성능을 높인 분산 처리 프레임워크     

## Spark Session? Context?
context는 spark driver가 cluster manager나 resoure manager와 communication 하고 spark의 기능들을 활용하기 위한 채널의 역할을 한다    
2.0 이후에는 상위의 개념인 session이 등장하여 spark context와 session 관련 정보를 담고있다.   
![0_XwWZAbmHF75xWRDR](https://user-images.githubusercontent.com/13589283/150999597-6b22dc10-29ee-4b44-b7b1-129167198460.png)


## 동작 과정
spark driver가 spark 전체의 main 함수를 실행하며, input을 task로 나누어 executor가 실행할수 있도록 함    
Input을 HDFS에서 읽어 오거나, 로컬에서 파일을 readFile 메소드를 이용하여 읽을 경우 자동으로 parallelize되며, 그외에 Collection 타입의 데이터로 변환이 필요한 경우 Spark Context의 parallelize 함수를 이용하여 Collection 타입으로 변환            
즉 spark-submit을 통해 job을 수행하면, spark driver는 메인 함수를 수행하며 SparkContext를 생성한다. SparkContext는 Cluster Manager와 연결되고 spark driver는 Cluster Manager로부터 executor 실행을 위한 리소스를 요청. SparkContext는 job을 task로 분할하여 executor들에게 보내고, 각 executor들은 task를 수행하고 결과를 저장한다.
