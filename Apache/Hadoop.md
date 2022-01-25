## Hadoop이란?
분산 환경에서 대용량 데이터의 처리를 지원하는 병렬 프레임워크


## Map / Reduce 연산 모델
사용자나 시스템이 정한 task들에 대해 Map / Reduce 2개의 phase로 연산을 하는 수행 모델     
(Input -> Splitting -> Mapping (중간 결과물 key, value 생성)-> Shuffling -> Reducing -> Final Result)    
Input은 파일을 load, split을 통해 task (rdd)로 쪼개며, mapping을 통해 task를 key-value 형태로 변환, 셔플링을 통해 key 별로 task를 리파티셔닝, reduce에서 사용자가 정의한 연산을 통해 최종 결과 도출     

![Map-only](https://user-images.githubusercontent.com/13589283/150996036-c6c2b5e8-8eee-42fc-91e6-51996713a5ef.png)


