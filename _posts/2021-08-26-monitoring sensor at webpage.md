---
title: "실시간으로 센서 데이터 웹페이지에서 확인하기"
excerpt: "AWS Amplify, API Gateway, Lambda 이용 DynamoDB에서 가장 최근 값 가져오기"
date: 2021-08-26
categories:
  - SensorMonitoring
tags:
  - AWS_Amplify
  - AWS_IoT
  - RPI
  - Tutorial
---

이 글은 [AWS IoT를 이용하여 센서 값 모니터링하는 IoT 시스템 구축 프로젝트](https://dongwon18.github.io/categories/#sensormonitoring)의 다섯 번째 글입니다.

안녕하세요. [이전 글](https://dongwon18.github.io/iot/get-DynamoDB-data-from-Web/)에서 Amplify로 DynamoDB의 값을 읽어오는 실습을 진행했습니다. 이번에는 해당 내용을 활용하여 실시간으로 데이터를 읽어오는 부분을 구현하려고 합니다.

전체 아키텍처 중 초록색 부분입니다. (초반 Mobile Hub를 이용하여 안드로이드 앱을 만드는 부분을 Amplify를 이용한 웹페이지 제작으로 변경하였습니다.)

<p align = "center">
  <img src = "/assets/images/프로젝트5.png" alt = "total architecture and implemented part"> <br/>
  전체 아키텍처 및 구현한 부분
</p>

# 목표

- DynamoDB에 **가장 최근에 저장된 센서 값**을 웹페이지를 통해 확인할 수 있다.
- 디바이스 ID를 검색하면 해당 디바이스의 가장 최신 데이터를 확인할 수 있다.
- 같은 네트워크 뿐 아니라 **다른 네트워크에 디바이스가 연결**되어 있어도 모니터링이 가능하다.
- **최소한의 보안**을 보장해야 한다.
- 구현은 Amazon 서비스를 이용한다.

# 사전 정보

이전 글에서 AWS Amplify를 이용하여 웹페이지를 기본적으로 만든 것에 덧붙여 만들 계획입니다. 센서 값 가져오기 등의 설정은 해당 프로젝트의 이전 글과 동일하게 진행했고 변경된 부분은 밑에 정리해두었습니다. 

- [AWS Amplify 이용 DynamoDB 데이터 웹페이지에서 검색](https://dongwon18.github.io/iot/get-DynamoDB-data-from-Web/)
- [AWS IoT를 이용하여 센서 값 모니터링하는 IoT 시스템 구축 프로젝트](https://dongwon18.github.io/categories/#sensormonitoring)

# 사용한 AWS

- AWS Amplify
- API Gateway
- AWS Lambda
- AWS IoT Rule
- AWS DynamoDB

# 전체적인 흐름

1. AWS Amplify를 이용하여 만든 웹페이지에서 센서 값을 알길 원하는 `Device ID` (이전 글에서 MQTT 주제 게시 및 DynamoDB에 저장할 때는 `client ID` 로 세팅한 값입니다.)를 입력 받습니다.
2. `Refresh` 버튼이 눌리면 API Gateway를 이용하여 요청(device id를 포함한 json 파일)을 Lambda에 전달합니다.
3. Lambda에서 현재 시간을 기준으로 일정 시간(저는 10초로 설정했습니다.) 이전 ~ 현재 시간까지 저장된 데이터를 불러와 가장 최근 데이터를 선택합니다.

    Lambda 함수에서 DynamoDB로부터 받은 `response`는 아래와 같은 json 형식입니다.(get_item을 이용하여 1개의 데이터만 찾아낸다면  `Items` 가 아닌 `Item` 입니다.)

    ```json
    response{
    	'Items': [{
    			'sensor_data': {
    				'temperature': Decimal('24'), 
    				'humidity': Decimal('48.099998474121094')
    			}, 
    			'store_time': Decimal('1629782758184'), 
    			'client_Id': 'RPI'
    		}, {
    			'sensor_data': {
    				'temperature': Decimal('24.200000762939453'), 
    				'humidity': Decimal('49.20000076293945')
    			}, 
    			'store_time': Decimal('1629782763733'), 
    			'client_Id': 'RPI'
    		}], 
    	'Count': 3, 
    	'ScannedCount': 3, 
    	'LastEvaluatedKey': {
    			'store_time': Decimal('1629782769248'), 
    			'client_Id': 'RPI'
    		}, 
    	'ResponseMetadata': {
    		'RequestId': 'HNC3P5GACNQS3F0LPIP3LLCHQVVV4KQNSO5AEMVJF66Q9ASUAAJG', 
    		'HTTPStatusCode': 200, 
    		'HTTPHeaders': {
    			'server': 'Server', 
    			'date': 'Tue, 24 Aug 2021 06:21:35 GMT', 
    			'content-type': 'application/x-amz-json-1.0', 
    			'content-length': '586', 
    			'connection': 'keep-alive', 
    			'x-amzn-requestid': 'HNC3P5GACNQS3F0LPIP3LLCHQVVV4KQNSO5AEMVJF66Q9ASUAAJG', 
    			'x-amz-crc32': '2271479515'
    		}, 
    		'RetryAttempts': 0
    	}
    }
    ```

4.  Lambda에서 store time, temperature, humidity를 statusCode 200과 함께 응답으로 보냅니다.(만약 해당 시간대에 값이 DynamoDB에 저장되어 있지 않다면 statusCode 204를 보냅니다.) 

    Lambda에서 보내는 응답은 코드 번호에 따라 다음과 같습니다.

    ```json
    {
    	'statusCode': 200,
    	'body':{
    		'time': '2021-08-25 14:00:00',
    		'temp': '24.000',
    		'humidity': '48.150'
    	}
    }

    {
    	'statusCode': 204
    }
    ```

5. 받은 응답에 담긴 내용을 화면에 출력합니다. 

# 튜토리얼

## 이전 글과 바뀐 점

이전 글과 방법 자체는 비슷해서 바뀐 점 위주로 설명해보겠습니다.

1. DynamoDB  key 구조
    - 이전에는 `timestamp`가 `partition key`이고 `client ID`가 `sort key` 였습니다. 그런데 웹페이지에서 실시간 데이터를 확인할 때는 client 별로 확인하는 것이 더 편리할 것이라고 생각이 되어 `partition key`를 `client ID`, `sort key`를 `timestamp`로 변경하였습니다.    
    > 개인적인 생각으론 데이터베이스 이론을 배울 때 sort key는 숫자일수록 처리에 유리하다고 배웠기 때문에 DB 측면에서도 효율적이지 않을까 합니다.    
    또한 DynamoDB에서는 partition key에는 =만 사용 가능하고 sort key에는 비교, oo로 시작 등을 사용할 수 있기 때문에 DynamoDB에서 효율적으로 가장 최근 데이터를 가져오기 위해서도 key 구조를 변경할 필요가 있었습니다.
    - DynamoDB Table을 한 번 생성하고 나면 key를 변경할 수 없습니다.(아무래도 이미 존재하는 데이터에 대해 key를 변경하면 처리하기 힘들기 때문일 거라 예상합니다.) 따라서 새로운 DynamoDB 테이블을 만들었습니다.(이름: Sensor_data)
    - 테이블을 새로 만들었기 때문에 Lambda 함수에서 **실행 역할의 table arn을 변경**해줘야 합니다.
2. Lambda 함수
    - 이전에는 timestamp와 client id를 API 요청에서 넘겨 받았는데 client id만 넘겨 받도록 변경했습니다.
    - 함수를 실행하는 시간을 기준으로 `일정 시간 전 ~ 함수 실행 시간`까지 DynamoDB에 저장된 데이터 중 가장 첫 번째 데이터를 읽어오도록 했습니다. (DynamoDB Query Key between 사용)
    - 데이터가 저장된 시간(한국 시간 기준), 온도, 습도를 정상적으로 데이터를 받았을 때 보내도록 했습니다.
    - 해당 시간대에 DynamoDB에 저장된 값이 없다면 204 status code를 보내 front end에서 에러를 구별할 수 있도록 했습니다.
3. HTML 
    - 이전에는 alert로 받은 응답을 json 형식 그대로 확인하도록 했으나 버튼 밑에 받은 값을 예쁘게 표시할 수 있도록 변경했습니다.
    - status code마다 출력하는 내용이 다르도록 설정했습니다.
        - 200: 받은 데이터 출력
        - 204: DynamoDB에 해당 값이 없다는 메시지 출력
        - 그 외: 지정되지 않은 code라고 출력(device id를 입력하지 않을 때 등 발생)

## 상세 구현

1. DynamoDB에 key 구조를 변경한 새로운 table인 Sensor_data 테이블을 만듭니다.
2. Sensor_data 테이블의 arn을 Lambda 함수에 연결된 역할에 입력합니다.
3. AWS IoT Rule RPI_data_store에서 데이터를 저장하는 테이블을 Sensor_data로 변경해줍니다.
4. Lambda 함수를 다음과 같이 변경합니다.

<script src="https://gist.github.com/dongwon18/93820154b84a084df0e452e7ebf00b98.js"></script>

  > 참고로 DynamoDB에서는 상위 5개 데이터 가져오기 등을 바로 실행하는 방법이 없습니다. 찾아보니 데이터를 저장할 때 번호를 매겨 해당 column을 기준으로 정렬하게 한 후 정렬된 값 중 번호가 일정 범위 이상인 데이터를 가져오는 식으로 쿼리를 작성하여 실행할 수 밖에 없다고 하더라고요. 제가 만든 테이블의 경우 timestamp를 포함하기 때문에 새로운 column을 만들지는 않고 timestamp를 활용해보기로 했습니다.

  ```python
    start = now - timedelta(seconds = 10) #now라는 시간보다 10초 전 시간을 지정합니다.

    table.query(
    	Limit = 3 # 조건에 부합하는 데이터가 많아도 최대 3개까지만 가져오도록 합니다.
    	KeyConditonExpression = #이 자리에 조건을 명시하면 됩니다.
    )
   ```

5. github의 `index.html` 을 다음과 같이 변경합니다.   
어떤 데이터인지, 단위는 무엇인지를 함께 명시하여 데이터를 확인할 때 보다 쉽게 확인할 수 있도록 했습니다.

<script src=[https://gist.github.com/dongwon18/2dadb7ca77f40192b376141259d6a8cc.js"></script>

6. `AWS Amplify 콘솔 -> 액세스 제어 -> 액세스 관리` 에서 페이지에 접속할 때 ID와 패스워드를 입력해야 접속할 수 있도록 설정할 수 있습니다.
7. 깃허브의 코드를 변경하면 AWS Amplify에서 배포가 완료될 때까지 기다립니다. 

## 결과

- RPI에서 sensor에서 값을 가져오는 코드를 실행하고 있는 상태에서 `Refresh` 를 눌렀을 때
    - refresh를 누르는 기준으로 가장 최근 값을 화면에 표시합니다. 여러 번 누르면 그 때마다 가장 최근 값을 표시합니다.

<p align = "center">
  <img src = "/assets/images/WEB_data.PNG" alt = "web page searching result"> <br/>
  웹페이지 검색 결과
</p>

- RPI에서 코드를 실행하지 않고 `Refresh` 를 눌렀을 때
    - 해당 시간부터 10초 전까지 저장된 데이터가 없다면 204 status code를 응답으로 받으므로 `Current data not exists in DynamoDB` 라고 출력합니다.
- client id 칸에 아무것도 작성하지 않고 `Refresh` 를 눌렀을 때
    - 적절한 요청을 주지 않았으므로 `Undefined statuscode undefined` 를 출력합니다.

# 결론

- AWS Amplify, API Gateway, Lambda 함수를 이용하여 DynamoDB에서 실시간으로 데이터를 가져와 확인할 수 있게 되었습니다.
- 웹페이지에 로그인을 하도록 함으로써 최소한의 보안을 적용할 수 있게 되었습니다.

# 참고

- [AWS DynamoDB developer Guide](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/GettingStarted.Python.04.html)(query 작성법 참조)
- [Boto3 Document](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html)(query 작성법 및 response 구조 참조)
