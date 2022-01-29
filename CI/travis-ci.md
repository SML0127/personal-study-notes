## Travis-ci?
웹을 통해 제공되는 CI(continuous integration) tool.   

## 사용법
예제 repository: https://github.com/SML0127/pse-extension     

1. github으로 로그인

2. repository(https://github.com/SML0127/pse-extension) 등록   

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

