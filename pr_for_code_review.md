원글: https://support.atlassian.com/bitbucket-cloud/docs/use-pull-requests-for-code-review/

p.s bitbucket에 디펜던시가 있는 내용(e.g. bitbucket에서의 pull request 확인 방법) 관련 내용은 제외

## Use pull requests for code review
우리는 파일 추가, 기존 코드의 업데이트 이후 merge를 수행 한다. 이때 우리는 업데이트 이후에도 코드의 퀄리티가 유지되며 기존 특징들을 해치지 않는 것을 보장하고 싶다. 코드 업데이트 및 향상을 위한 피드백을 얻기 위해 추가한 모든 코드들을 포함해서 pull request를 생성할 수 있다. pull request는 동료들에게 코드 리뷰 요청 및 가장 최근 커밋을 기준 빌드 상태 검사 방법(method)을 제공한다.          


## PR request process
코드 리뷰와 협업(collaboration)은 pull request의 핵심이다. 너의 역할에 따라 하나 이상의 pull request들에 대해 작성자(author), 검토자(reviewr) 또는 둘 다 일 수 있다. 아래는 end-to-end pull request 과정이 어떻게 동작하는지에 대한 삽화(illustration)이다.            
<img width="589" alt="PR_review_process" src="https://user-images.githubusercontent.com/13589283/155733287-1612a764-165e-48b9-b121-e60440baeb99.png">
       
       
## Pull request authors

