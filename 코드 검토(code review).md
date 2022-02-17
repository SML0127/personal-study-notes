## 코드 검토(code review)?

단순히 코딩 스타일, 개발 표준같은 가이드라인에 따른다는 것이 아니다.        
peer code review라고도 하며 서로 간의 코드를 보며 중복, 에러, 사이드이펙트, 유지보수의 관점에서 수정해야할 것들을 점검하고 반영하는 것이다. 이를 통해 개발 프로세스의 가속화 및 간소화가 가능하다.     


## 어떤 것들을 체크?
1. 기본적인 기능과 테스트 코드의 동작 여부
2. 버그, 에러, 사이드 이펙트 등등
3. 가독성, 유지보수가 쉬운지
4. 중복 여부, 재사용 가능한 부분은 없는지
5. 개발 표준, 가이드라인을 잘 따르는지
6. 코드를 보고 배울점은 없는지     


## 코드 리뷰를 받는 자세
코드 리뷰하는 사람이 쉽게 알아 볼수 있도록 코드에 대한 충분한 정보를 제공하는 것이 중요하다. 


## 코드 리뷰 하는 자세
어떤 점들을 어떻게, 왜 수정하였는지에 대해 공유가되어야한다. PR 템플릿을 많이 활용해보자.


아래 코드들을 리뷰한다 하였을때
~~~
## db-server.py
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

코드 리뷰 이후의 버전
~~~
## db-server.py
// connection과 cursor 관리
cur = conn.cursor()


// table과 column 변수화 (local variable -> global variable로 변경)
table_task = "task"
table_stage = "stage"
column_task_input = "input"
column_stage_level = "level"
column_task_id = "id"
column_task_stage_id = "stage_id"

// 변수 명 재사용
get_input_level_query = "select %s.%s, %s.%s from %s join %s on %s.%s = %s.%s where %s.%s = %s;".format(table_task, column_task_input, table_stage, column_stage_level, table_task, table_stage, table_stage, column_task_id, table_task,column_task_stage_id,  table_task, column_task_id)
cur.execute(get_input_level_query % str(task_id))
(input_url, level) = cur.fetchone()

// table과 column 변수화 (local variable -> global variable로 변경)
table_succeed_task_detail = "succeed_task_detail"
column_std_output = "output"
column_std_task_id = "task_id"

// 변수 명 재사용
get_output_query = "select %s from %s where %s = %s;".format(column_std_output, table_succeed_task_detail, column_std_task_id)
cur.execute(get_output_query % str(task_id))
output_url_list = cur.fetchall()[0]

conn.commit()

// 변수 명 재사용
db_cur = db_conn.cursor()
~~~

~~~
## CrawledPage.react.js 
get_latest_progress(){
  const obj = this;
  axios.post(setting_server.DB_SERVER+'/api/db/executions', {
    req_type: "get_latest_progress",
    job_id: obj.props.JobId,
  })
  .then(function (response) {
    if (response['data']['success'] == true) {
      console.log(response)
      obj.setState({
        current_detail_num: response['data']['result'][0],
        expected_detail_num: response['data']['result'][1], 
        progress_detail: isNaN(parseFloat(response['data']['result'][1]) / parseFloat(response['data']['result'][0]) * 100 ) ? 0 : (parseFloat(response['data']['result'][1]) / parseFloat(response['data']['result'][0]) * 100 )
      })
    } 
  })
  .catch(function (error){
    console.log(error);
  });
}
~~~

1. obj라는 변수의 변수명을 더 목적이 들어나도록 (가독성)
2. result안의 값들을 여러번 가져다 쓰니 미리 변수로 저장하자 (재사용성)
3. 계산 식은 따로 빼서 더 정리하자 (가독성)
4. '/api/db/executions' 부분도 변수로 (유지보수)


코드 리뷰 이후의 버전
~~~
## CrawledPage.react.js 
get_latest_progress(){
  // 변수명 목적이 들어나게
  const this_obj = this;
  
  // 유지보수를 위해 변수화 (local to global)
  api_execution = '/api/db/executions'
  
  axios.post(setting_server.DB_SERVER + api_execution, {
    req_type: "get_latest_progress",
    job_id: this_obj.props.JobId,
  })
  .then(function (response) {
    if (response['data']['success'] == true) {
      console.log(response)
      
      // 중복 사용되는 것들 변수화
      local_current_detail_num = response['data']['result'][0]
      local_expected_detail_num = response['data']['result'][1]
      
      // 계산식은 빼서
      local_progress_detail = isNaN(parseFloat(local_expected_detail_num) / parseFloat(local_current_detail_num) * 100 ) ? 0 : (parseFloat(local_expected_detail_num) / parseFloat(local_current_detail_num) * 100 )
      
      obj.setState({
        current_detail_num: local_current_detail_num,
        expected_detail_num: local_expected_detail_num, 
        progress_detail: local_progress_detail
      })
    } 
  })
  .catch(function (error){
    console.log(error);
  });
}
~~~
