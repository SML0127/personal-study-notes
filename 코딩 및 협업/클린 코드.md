원글:https://garywoodfine.com/what-is-clean-code/, https://medium.com/swlh/what-is-clean-code-463d25fa6e0b

## Bad code?
Bad code 계속 쌓일 수록 개발자들의 유지보수는 더 힘들어지고, 이는 개발자들의 생산성이 떨어지는 결과를 초래한다. 생산성 감소를 경영진이 알게되면 엉망진창(mess)을 해결하기 위해 더 많은 자원을 추가하고 결국 상황을 악화 시킬 것이다.       
![1_1iVzguwwwvS9DJcsZK4kOg](https://user-images.githubusercontent.com/13589283/156587273-37a08d86-5c46-45c9-b862-3b75fe1b3164.png)        
데드라인에 맞추기위해 엉망진창(mess)를 만드는데, 중요한점은 한번 엉망진창(mess)를 만들면 이는 즉시 너의 생산성을 떨어뜨리기 시작한다. 데드라인에 맞추기 위해, 항상 코드를 클린하게 유지해야 한다.



## 클린 코드(Clean code)?        
클린 코드의 가장 유명한 정의는 다음과 같다.     
<div align="center">
  <h4> 클린 코드란 이해하기 쉽고 수정하기 쉬운 코드이다.(Clean code is code that is easy to understand and easy to change)</h4>
</div>
       
그 외에 클린 코드라면 아래의 특징들을 가진다.
- 하나의 일을 계속해서 수행 하지 않는다. (Don't do one thing over and over again)
- 코드를 보면 코드를 작성한 사람이 코드를 케어(care)하고 누가 읽고/유지하는지 알수있다. (It shows the person who wrote the code really ‘Cares’ for the code and who is reading/maintaining it)
- 로직이 직관적이고 버그를 발생할 여지가 없다. 다른 누군가가 코드를 악화시키도록 유도 하지 않는다. (The logic is straight forward and doesn’t leave room for bugs to creep up. It doesn’t tempt others to modify it which will make the code worse)
- 각 함수, 클래스들은 주변을 둘러싸는 클래스/함수들과 완전히 독립적인 책임을 나타낸다. (Focused, each function, classes exhibits one responsibility that they have which remains completely independent off the surrounding classes/functions)
- 해결해야할 문제의 텐션을 드러내고, 긴장을 극에 달하게한 후 독자에게 긴장을 해결하는 순간을 선사한다. (Code exposes the tension in the problem to be solved, builds the tensions to climax and then gives the reader ‘Aha!’ moment as the tensions are resolved)
- 유닛 테스트를 반드시 가져야한다.
- 특정 문제를 풀기위해 프로그래밍 언어가 만들어진것 처럼 보여야한다. (The code makes the programming language looks like it was made to solve the particular problem) 


## 이해하기 쉬운 코드?
코드를 이해 한다는 것은 아래의 의미들을 지니며 클린 코드라면 각각을 쉽게 이해할 수 있어야 한다.    
- 전체 수행 흐름(flow)의 이해
- 서로 다른 각각의 오브젝트들이 어떻게 서로 동작하는지 이해
- 각 클래스들의 역할과 책임에 대한 이해
- 각 메소드들이 하는일에 대한 이해
- 각 표현과 변수들의 목적에 대한 이해 
