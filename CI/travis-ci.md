## [Travis-ci](https://www.travis-ci.com/)?
![ci](https://user-images.githubusercontent.com/13589283/151662805-e48ded2f-8f49-4845-9e2d-5267809c36d9.jpeg)        
웹을 통해 제공되는 CI(continuous integration) tool.   

## 사용법
예제 repository: https://github.com/SML0127/pse-extension     

1. github으로 로그인

2. repository(https://github.com/SML0127/pse-extension) 등록   
![image](https://user-images.githubusercontent.com/13589283/151662635-9ff1d3a6-e57c-40cd-a611-0113e3a53ee6.png)

3. .travis.yaml을 작성 및 repository에 추가   
~~~
language: node_js
os: osx
node_js:
  - 14.17

branches:
  only:
    - master

before_install:
  - npm update
  - npm install 

install:

script:
  – npm run build
~~~

4. setting에서 빌드 조건 선택
![image](https://user-images.githubusercontent.com/13589283/151662665-d504ef2b-a3fa-4e96-9d81-cd02c0c28d14.png)


5. repository에 push 하여 자동으로 빌드 및 성공하는지 확인
![image](https://user-images.githubusercontent.com/13589283/151662748-de8f2cb6-e860-4e4d-a5a8-bd2fe8e0cd00.png)


