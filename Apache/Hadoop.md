## Hadoop이란?
분산 환경에서 대용량 데이터의 처리를 지원하는 병렬 프레임워크


## Map / Reduce 연산 모델
사용자나 시스템이 정한 task들에 대해 Map / Reduce 2개의 phase로 연산을 하는 수행 모델   
Map은 slave node들에 대해 parallel하게 연산 수행하며 인풋으로 받은 task를 key, value 형태로 변환하고, 연산의 (중간)결과에 대해 Reduce를하여 다음 연산을 수행   

<img width="830" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/13589283/150994463-9ce8cc0b-4755-4f2d-83eb-3dd4244d5dae.png">
