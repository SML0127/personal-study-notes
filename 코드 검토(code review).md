출처: https://smartbear.com/learn/code-review/what-is-code-review/
## 코드 검토(code review)?

단순히 코딩 스타일, 개발 표준같은 가이드라인에 따른다는 것이 아니다.        
peer code review라고도 하며, 서로 간의 코드를 보며 중복, 에러, 사이드이펙트, 유지보수의 관점에서 수정해야할 것들을 점검하고 반영하는 것이다. 이를 통해 개발 프로세스의 가속화 및 간소화가 가능하다.     


## 어떤점들을 확인헤야하나
1. 기본적인 기능과 테스트 코드의 동작 여부
2. 버그, 에러, 사이드 이펙트 등등
3. 가독성, 유지보수가 쉬운지
4. 중복 여부, 재사용 가능한 부분은 없는지
5. 개발 표준, 가이드라인을 잘 따르는지
6. 코드를 보고 배울점은 없는지     




아래 코드(db-server.py)를 리뷰한다 하였을때
~~~
            cur = conn.cursor()
           
            query = "select task.input, stage.level from task join stage on stage.id = task.stage_id where task.id = %s;"
            cur.execute(query % str(task_id))
            (input_url, level) = cur.fetchone()
          
            query = "select output from succeed_task_detail where task_id = %s;"
            cur.execute(query % str(task_id))
            output_url_list = cur.fetchall()[0]

            conn.commit()
            cur = db_conn.cursor()
~~~            

1. 변수명을 지을때 더 목적이 들어나도록 (가독성)
2. query 라는 변수의 재사용 제거 (가독성, 재사용성)
3. db connection의 관리 (버그, 에러, 사이드 이펙트)
4. table / column등의 이름이 바뀔 수 있으니 변수로 관리 (유지보수)
5. 테스트 코드의 추가 (테스트 코드) 

