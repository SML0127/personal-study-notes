## Apache Druid란?   
큰 데이터들에 대해 OLAP 질의(slice and dice)를 빠르게 수행하기 위해 고안된 실시(low latency)간 분석용 데이터베이스이다. (A real-time analytics database designed for fast slice-and-dice analytics ("OLAP" queries) on large data sets)   

## 특징
 - Columnar storage format: 질의 수행시 필요한 컬럼들만을 로드하며 이를 통해 빠른 스캔, 랭킹, groupBy를 지원     
 - Native search indexes: inverted index를 통해 string 값들에 대한 빠른 검색 및 필터 지원     
 - Streaming and batch ingest: Kafka, HDFS, AWS SE등의 스트리밍 커넥터를 지원     
 - Flexible schemas: 변화하는 schema와 nested data들에 대한 지원     
 - Time-optimized partitioning: 시간을 기반으로 데이터를 파티셔닝하며, 시간 관련 질의(time-based query)를 기존의 database보다 빠르게 지원     
 - SQL support: JSON 기반 언어외에도 HTTP, JDBC를 통해 SQL 사요을 지원
 - Horizontal scalability: 초당 수백만개의 이벤트를 수집하고, 수년간의 데이터를 유지하며 sub-second 질의(Fast Interactive queries which can be used to power interactive dashboards, fast analytics, monitoring and alerting applications)를 지원

## Integration
카프카, 하둡, 플링크등의 data source들과 연동을 지원   
Storage layer와 end-user layer 사이에 query layer로써 analytics workloads를 수행하기위해 존재
![diagram-3](https://user-images.githubusercontent.com/13589283/150742478-e1ed9e73-b2b1-4aa3-9641-19ce135f8524.png)


## Ingestion
스트리밍(e.g. Kafka)과 배치(e.g. HDFS) ingestion을 지원   
raw data를 read에 최적화된 format인 Druid segement(read-optimized format)로 바꾸는 인덱싱을 지원  
![diagram-4](https://user-images.githubusercontent.com/13589283/150743078-5d3bc427-75cd-4359-b16c-c6f16a76259e.png)

![1024px-Druid_Open-Source_Data_Store,_architecture,_DruidArchitecture3 svg](https://user-images.githubusercontent.com/13589283/150739968-de620cb1-0da9-4393-9477-81486170c24d.png)

= OLAP = online analytical processing 
 = 데이터들을 미리 분석 해 놓는 database다 = 즉 기존 RDB들은 데이터들을 저장하는것이 주 목적이라면 드루이드는 그 데이터들의 분포, 경향, 어그리게이션등을 인덱스로써 업데이트해가며 유지 = 
타임스탬프, 디멘션, 메트릭
