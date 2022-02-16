출처: https://smartbear.com/learn/code-review/what-is-code-review/
## 코드 검토(code review)?

단순히 코딩 스타일, 개발 표준같은 가이드라인에 따른다는 것이 아니다.        
peer code review라고도 하며, 서로 간의 코드를 보며 중복, 에러, 사이드이펙트, 유지보수의 관점에서 수정해야할 것들을 점검하고 반영하는 것이다. 이를 통해 개발 프로세스의 가속화 및 간소화가 가능하다.     


## 어떤점들을 확인헤야하나
1. 기본적인 기능과 테스트 코드의 동작 여부
2. 버그, 에러, 사이드 이펙트 등등
3. 가독성, 유지보수가 쉬운지
4. 개발 표준, 가이드라인을 잘 따르는지



아래 코드(db-server.py)의 예시
~~~
            cur = conn.cursor()
           
            query = "select task.input, stage.level from task join stage on stage.id = task.stage_id where task.id = %s;"
            cur.execute(query % str(task_id))
            (input_url,level) = cur.fetchone()
          
            query = "select output from succeed_task_detail where task_id = %s;"
            cur.execute(query % str(task_id))
            output_url_list = cur.fetchall()[0]

            conn.commit()
            cur = db_conn.cursor()
~~~            

다른사람들이 해당 코드를 본다 하였을때 고쳐야할 점들을 리스트업 해보면
1. 변수명을 지을때 좀더 목적이 들어나게 지어야함
2. query 라는 변수의 재사용 제거
3. db connection의 관리
4. table / column등의 이름이 바뀔 수 있으니 변수명으로 관리

