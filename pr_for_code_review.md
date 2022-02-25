원글: https://support.atlassian.com/bitbucket-cloud/docs/use-pull-requests-for-code-review/

P.S. bitbucket에 디펜던시가 있는 내용(e.g. bitbucket에서의 pull request 확인 방법)은 제외

## Use pull requests for code review
우리는 파일 추가, 기존 코드의 업데이트 이후 merge를 수행 한다. 이때 우리는 업데이트 이후에도 코드의 퀄리티가 유지되며 기존 특징들을 해치지 않는 것을 보장하고 싶다. 코드 업데이트 및 향상을 위한 피드백을 얻기 위해 추가한 모든 코드들을 포함해서 pull request를 생성할 수 있다. pull request는 동료들에게 코드 리뷰 요청 및 가장 최근 커밋을 기준 빌드 상태 검사 방법(method)을 제공한다.          


## PR request process
코드 리뷰와 협업(collaboration)은 pull request의 핵심이다. 너의 역할에 따라 하나 이상의 pull request들에 대해 작성자(author), 검토자(reviewr) 또는 둘 다 일 수 있다. 아래는 end-to-end pull request 과정이 어떻게 동작하는지에 대한 삽화(illustration)이다.            
<img width="589" alt="PR_review_process" src="https://user-images.githubusercontent.com/13589283/155733287-1612a764-165e-48b9-b121-e60440baeb99.png">
       
       
## Pull request authors
pull request의 작성자로써, 코드 리뷰 프로세스는 pull request를 검토자들에게 생성함으로써 공식적으로 시작된다. pull request를 생성하고 검토자를 추가한 후, 승인을 기다리는 동안 휴식을 취하는 경향(inclined to)이 있다. 그러나 검토자가 코드를 보고 코멘트를 작성하면 pull request에 진행중인 토론에 대한 이메일 알람을 받으며, 응답할 기회를 제공하고 우리는 코드 리뷰 프로세스의 적극적으로 참여(active participant)하게 된다.

## Pull request reviewrs 
코드 리뷰를 진행하며 피드백, 제안, 아이디어를 코멘트 할것이다. 명백한 로직 에러(obvious logic error) 여부, 모든 경우가 완벽히 구현되었는지 여부, 현 자동화된 테스트들을 다시 작성해야할 필요가 있는지 여부, 그리고 코드가 현 스타일 가이드라인을 따르는지 여부등을 확인하는데 시간을 소모할 수 있다. 리뷰 이후, pull request가 merge될 준비가 되었가나 작성자가 너의 코멘트들을 해결할 수 있다 신뢰하면 승인을 한다.
