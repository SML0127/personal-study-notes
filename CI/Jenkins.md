## [Jenkins](https://www.jenkins.io/)란?
![Jenkins_logo_with_title svg](https://user-images.githubusercontent.com/13589283/151704948-0469a83f-a0f7-4485-8f96-3c6c5e27e49a.png)       
소프트웨어의 build, test, delivering 또는 deploying와 관련된 모든 종류의 작업을 자동화하는데 사용할 수 있는 독립형 오픈 소스 자동화 서버이다. (Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.)

## 셋업 
기준: 맥 os  
예제 [repository](https://github.com/SML0127/pse-extension)   

1. brew를 통한 jenkins 설치 jenkins            
   brew install jenkins

2. jenkins 계정 설정     

3. 필요 plugin 설치 및 설정     
   - NodeJs 설치 및 global tool configuration에서 node js 설정 (설정 이름은 pse-node-js).        
   
4. Job 생성 및 git repository 설정     

5. Jenkinsfile 작성 및 repository에 등록
~~~
pipeline {
    agent any
    tools {nodejs "pse-node-js"}
    stages {
        stage('Build') {
            steps {
                sh '''
                echo "Start Build"
                npm run build
                '''
            }
        }
    }
}
~~~

6. git에 push 하여 빌드 확인

