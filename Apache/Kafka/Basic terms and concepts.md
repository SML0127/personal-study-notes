
### Kafka cluster
 - 프로듀서 (Producer)

레코드를 생성 → 프로듀서 파티셔너(producer partitioner)가 레코드의 key에 따라 어느 leader partition으로 보낼지 결정
→ 매칭되는 topic의 leader partition으로 전송 
key 없으면 내부적으로 처리해서 할당 (지금 이 이상 알 필요는 없음)
partition num =  murmur2 algorithm을 사용하여 hash 생성 + # of partitions으로 나누기


 - 컨슈머 그룹(Consumer groups)
모든 토픽의 파티션들을 컨슈머 그룹의 컨슈머들이 나누어 할당하여 담당
컨슈머가 없어지거나 새로 생기면 re-assign되며, 잠재적으로 컨슈머들은 파티션들을 공유하게됨
topic들로부터 데이터를 가져오며, 파티션 별로 commited offset을 관리하여 어디까지 가져왔는지, lag 계산등에 사용



 - 주키퍼 (Zookeeper)




 - 브로커 (Broker)

파티션들이 저장되어있는 서버
파티션 별로 leader broker, follower broker이 존재하며 leader의 역할은 replication을 follower broker에게 할당하는 것









Kafka에서 사용되는 저장 단위

Topic  >  Partition  >  Segment

 - 토픽(Topic)
데이터 관리를 위한 논리적 단위이다.
1개 이상의 partition으로 구성되며, 이미 추가한 partition은 삭제 불가하다.
→  partition을 줄이는 방법은 없고, 정 줄이고 싶다면 새로이 topic을 만들어야함 
(카카오 공용 kafka) -> 사용자(개발자)가 직접 신청해서 사용


 - 파티션(Partition)
프로듀서를 통해 레코드(= 메세지, 데이터..)를 저장, 컨슈머가 요청시 레코드(들)을 전달하는 단위
파일시스템에 저장되는 물리적 단위
leader partition과 set of replicas(= followers)로 구성
# of 파티션 >= 컨슈머, 보통 배수로 지정이 효율적
파티션의 크기 → 작으면 딜레이 발생 (처리속도 저하), 크면 자원 소모량이 커짐  
(카카오 공용 kafka) -> 사용자(개발자)가 직접 신청해서 사용


 - 세그먼트(Segment)
파티션을 구성하는 실제 파일 
log segment라고도 불리는듯 
segment에는 크기(log.segment.bytes)와 시간(log.roll.hours) 조건이 있고 둘중 하나라도 만족하면(default = 1GB, 7 days) 새로운 segment 생성 
현재 write가 일어나는 segment = active segment


레코드, 메세지, 데이터, 이벤트
세그먼트에 저장되는 레코드
레코드 = header(topic, partition, timestamp ..) + key + value
→  header는 topic,metadata등을 보관, value는 실제 data, key의 역할?
→ 동일 key = 동일 partition
   (why? partition의 개수를 늘리게되면 같은 key에 대해 늦게 들어온 message가 먼저 consume될 가능성 제거 = offset 보장)
 - Offset
 - Lag
마지막으로 produce된 메시지와 마지막 consumer committed offset의 차이
(즉, produce의 생산 속도 - consumer 처리 속도로 커질수록 생산에 비해 처리가 못따라가는 경우)
  
 - Replica
흔히 아는 그 replica
Partition에 대한 복제본
대다수 클러스터는 1 Leader, 1 Follower
(카카오 prod-pg2-kafka) → 1 Leader, 2 Followers
