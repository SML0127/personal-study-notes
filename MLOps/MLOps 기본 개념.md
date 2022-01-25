원글 출처: https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning   
참고글: https://jaemunbro.medium.com/mlops%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B3%A0-84f68e4690be   

## MLOps란?
  - 기존 DevOps의 개발과 운영을 합쳐 생산성과 운영의 안정성을 최적화하는 방법론을 ML에 적용한 방법론이다.
  - 머신러닝을 서비스에 적용하기 까지의 일련의 과정들에 대한 운용이라고 정의 하기도한다.
  - 데이터 수집, 전처리 및 분석하는 단계 (Data Collection, Ingestion, Analysis, Labeling, Validation, Preparation)와 모델을 생성, 학습 및 배포하는 단계(Model Training, Validation, Deployment) 전 과정을 하나의 라이프사이클로 본다.
  - 데이터 수집부터 모델을 만들어내는 것(test 환경)과 프로덕션 환경에 적용하는 것은 다른 레벨의 문제이다.(프로덕션 환경을 고려안하면 MLOps level 0)   



## ML system과 DevOps의 차이점

  1. Team skills   
    - ML project에는 data scientist, ML researcher등 SW 엔지니어링 경험이 덜 한 멤버가 포함된다. 

  2. Developement   
    - 파라미터, 알고리즘, 모델링 기법 등을 바꿔가며 어떤 것들이 효과가 있었고, 없었는지에 대해 트랙킹 및 재현이 가능 해야한다.

  3. Testing   
    - 기존의 유닛 / 통합 테스트 외에도 데이터, 모델, 학습된 모댈의 퀄리티등에 대한 평가가 필요하다.

  4. Deployment   
    - ML 시스템은 multi-steps 파이프라인에서 자동으로 재학습 및 모델을 사용하는것을 요함. 이런 파이프라인은 전체 과정을 복잡하게 만들며, 데이터 사이언티스트들의 배포 이전에 매뉴얼하게 수행되던 새로운 모델의 학습과 평가 단계를 자동화할 필요가 있음.
  
  5. Production   
    - 모델의 성능은 data profile로 인해 성능이 떨어 질 수 있다. 즉, 꾸준히 모니터링하며 모델의 성능이 떨어기는 것을 파악하고 expectation에서 벗어날 경우 롤백할 수 있어야함.


## ML system과 DevOps의 공통되나 조금 차이가 있는 것들

  1. CI   
    - 지속적 통합에는 코드, 컴포넌트 뿐만이 아니라 데이터의 테스팅, 검증, 데이터 스키마 모델등을 다 포함함.

  2. CD   
    - 서비스가 아닌 ML 파이프라인을 포함한 또는 시스템 레벨의 자동 배포. 

  3. CT   
    - 자동으로 모델 재학습 및 제공. 
  
  
  
## ML의 data science steps

  1. Data extraction   
    - 다양한 소스로부터 ML에 사용될 데이터 선택 및 통합.

  2. Data analysis   
    - ML 모델을 만들기위해 사용 가능한 데이터(e.g. 데이터 스키마, 특징)에 대한 광범위한 이해.
  
  3. Data preparation   
    - ML을 위한 data 준비. 이 과정에는 데이터를 학습, 평가, 테스트 셋으로 나누는 클리닝도 포함.   

  4. Model training   
    - 다양한 ML 모델을 학습하기위해 서로 다른 알고리즘들을 사용(output: trained model).

  5. Model evaluation   
    - test set을 이용하여 모델의 퀄리티 평가 (output: set of metrics to assess the quality of model)

  6. Model validation   
    - 모델 배포를 하기전 성능이 베이스라인 이상으로 잘 나오는지(배포에 적합한지) 검증.  

  7. Model serving   
    - 검증된 모델이 타겟 환경에서 사용되는것을 의미. 환경: REST API를 통해 / mobild device or edge device(데이터를 발생하는 기기) / batch prediction 시스템의 일부  

  8. Model monitoring   
    - 모델의 성능과 이후 iteration에서의 예상 성능 모니터링. 

## MLOps level 0: manual process
<img width="840" alt="lv0" src="https://user-images.githubusercontent.com/13589283/150079481-7547a17d-d667-4230-9019-75f96e398374.png">

  - 데이터의 수집부터 모델 생성, 평가 및 배포까지 모든 step이 매뉴얼하게 동작하는 레벨.   
  - 모니터링이 없는데, 배포된 이후는 모델에 관여 안함. 즉 CD가 없다.   
  - 실제 환경에서의 데이터 상태, 변화로 인해 모델 성능 저하 발생 가능.   

## MLOps level 1: manual process
<img width="840" alt="lv1" src="https://user-images.githubusercontent.com/13589283/150079505-bc3ad3fb-27ec-4c69-9102-354f4ed1ce61.png">

  - ML 실험에서 각 단계들이 자동화되어 빠른 iteration이 가능해지며 전체 파이프라인을 프로덕션으로 옮길 수 있는 readiness를 가짐 (Rapid experiment).   
  - Production에서도 모델이 계속 학습됨. 단 모니터링하며 성능이 예측보다 낮아지는지 파악 필요.    
  - 새로운 데이터로 모델이 학습되면 지속작으로 배포   
  - 모델 외에도 파이프라인도 배포됨.    
  - Level 0 에서는 매뉴얼하게 모델을 학습하고 이를 제공하는것, Level 1 에서는 파이프라인의 각 단계가 자동화되며 모델과 파이프라인이 같이 배포되고 배포된 모델의 성능을 계속 모니터링하며 모델이 지속적으로 학습 및 배포됨.   
  - Feature store라는 컴포넌트가 있는데, 모델 학습과 제공에 필요한 피처들이 저장되며 높은 쓰루풋의 배치 서빙과 저 지연성의 리얼타임 서빙의 API를 통해 제공해야함.  
  
## MLOps level 2: CI/CD pipeline automation   
<img width="840" alt="lv2" src="https://user-images.githubusercontent.com/13589283/150080330-c9428079-d108-4146-9d44-4904562355c3.png">

  - Level 1에서 CI / CD가 robust해진 버전   
  - 모델에 대한 빌드, 테스트, 파이프라인 패키징 등이 자동으로 통합되며(CI) 이렇게 패키징된 package가 지속적으로 배포(CD)됨   
  - 배포된 package를 이용하여 모델은 프로덕션 레벨에서 다시 학습되고, 모니터링됨. 또한 이러한 결과는 테스트 레벨에서의 from 데이터 처리 to CD 까지 영향을 줌 


