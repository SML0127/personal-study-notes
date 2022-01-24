출처: https://druid.apache.org/technology

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


## Storage
Column의 type에 따라 다른 압축, 인코딩 방법을 적용하며, column의 type 별로 적절한 인덱스 빌드 지원   
검색 시스템들과 유사하게 string에대해 inverted index를 빌드하여 검색과 필터를 지원    
시계열 데이터베이스와 유사하게 시간을 기준으로 데이터를 파티셔닝하여 time-oriented 질의를 빠르게 지원   
옵셔널하게 데이터가 들어오는 시점에 aggregate를 수행(roll-up)할 수 있고, 이를 통해 storage 공간을 절약 가능
![diagram-5](https://user-images.githubusercontent.com/13589283/150743899-4db5ff67-f19e-4402-bfda-8480a117b9f7.png)

## Query
JSON over HTTP(druid는 질의대신 JSON object를 인풋으로 받을 수 있음, 이를 객체 지향 쿼리 즉 native query라 함)나 SQL을 인풋으로 받으며, 외에도 druid만의 operator들로 counting, ranking등의 빠르게 지원
![diagram-6](https://user-images.githubusercontent.com/13589283/150744691-38b999f2-80b4-4e4a-be92-0cfea68d1681.png)

## Architecture
microservice-based 아키텍처로 각각의 코어 서비스들(ingestion, querying, coordination)등은 분리되거나 합쳐져서 상용 hardware에 배포될 수 있음   
모든 주요 서비스들에 네이밍을 하여 use case와 workload에 따라 각각의 서비스들에 대한 fine-tune(미세 조정)을 지원. 예를 들어 ingestion 서비스에 적은 resource를 주고 query 서비스에 더 많은 resource를 줄 수 있음   
서비스가 fail나더라도 다른 서비스에 영향을 주지 않고 독립적 처리 
![diagram-7](https://user-images.githubusercontent.com/13589283/150744714-e8d99315-62cc-499c-9397-48688d49ec39.png)


## Operations
 - Data replication: 데이터는  원하는 만큼 복제되어 단일 서버의 failure는 질의 수행에 영향을 주지 않음
 - Independent services: 주요 서비스들에 명시저으로 네이밍을 하여 각 서비스들을 미세 조정 할 수 있음. ingestion에 실패하면 새로운 데이터가 들어오지 않지만 그렇다고 기존 데이터는 질의 가능한 상태로 존재
 - Automatic data backup: 인덱스된 모든 데이터를 HDFS 같은 filesystem에 백업함으로 druid cluster 전체를 잃어도 빠르게 복원 가능
 - Rolling updates: 사용자의 서비스에 영향을 주지 않고 (끊김 없이) druid cluster를 업데이트 가능




## 개인적 생각
즉, 데이터를 저장하며 데이터들의 분포, 경항, aggregation등을 인덱스로 업데이트 해가며 유지 + druid cluster를 유지하여 단일 서버에서 failure가 나더라도 문제가 없도록 하는 것이 주요 장점 인듯? 데이터가 자주 업데이트되거나 테이블 간의 조인을 요하는 환경에는 부적합    
클러스터 구성을 보면 스트리밍 데이터를 받은 real-time node, 세그먼트의 관리 및 분산을 담당하는 coordinatior node, 질의를 받는 broker node, broker node로 받은 질의를 딥스토리에서 가져온 세그먼트에서 수행하여 그 결과를 반환해주는 historical node, 클러스터 조율에는 주키퍼, 메타데이터 스토리지는 RDB들이, deep storage는 HDFS나 AWS S3가(영구 데이터 백업용)으로 사용 
![1024px-Druid_Open-Source_Data_Store,_architecture,_DruidArchitecture3 svg](https://user-images.githubusercontent.com/13589283/150739968-de620cb1-0da9-4393-9477-81486170c24d.png)
