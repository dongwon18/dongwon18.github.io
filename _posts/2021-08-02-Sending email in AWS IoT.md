---
title: "AWS IoT에서 특정 조건일 때 메일 보내기"
excerpt: "AWS Lambda, SNS, AWS IoT Rule로 자동으로 메일 보내기"
date: 2021-08-02
categories: 
  - AWS_IoT
tags:
  - Tutorial
---

안녕하세요. AWS IoT에서 MQTT 프로토콜을 이용하여 디바이스로부터 받은 값을 이메일로 보내는 AWS IoT 자습서 내용을 소개해보려 합니다. 자습서에는 SMS 문자로 보내는 예제가 있었는데 이메일은 프리티어가 제공되기 때문에 이메일로 진행해봤습니다. (SMS는 통신사 규정에 따라 다르게 가격이 책정되더라고요.) 

# 사용한 AWS 서비스

- AWS IoT: Rule 사용
- Amazon SNS
SNS에서 이메일 보내기는 매달 1000개가 프리 티어로 제공됩니다.
- Amazon Lambda
요청 백만 건, 초당 400,000GB 매달 프리티어로 제공됩니다.

# 전체적인 흐름

지역을 서울로 선택하지 않는다면 MQTT 동작이 제대로 안 돼서 MQTT client 테스트 화면에서는 볼 수 있지만 메일은 오지 않는 현상이 발생합니다. 꼭 `서울` 로 region 선택했는지 확인하고 진행해주세요.

1. AWS IoT에서 MQTT로 디바이스에 데이터를 받는다.
2. 받은 데이터가 Rule에 명시된 조건을 만족하면 Lambda에 해당 데이터를 보낸다.
3. Lambda에서는 받은 데이터를 보기 예쁜 모양으로 가공한다.
4. 가공한 값을 SNS에 넘긴다.
5. SNS는 받은 값을 지정한 이메일 주소로 보낸다.
6. 이메일에서는 예쁘게 가공된 문자열을 확인할 수 있다. 
  - Lambda를 사용하지 않고 SNS와 Rule만 이용하여 데이터를 받는다면 밑 사진처럼 json 파일 형식 그대로 나오게 됩니다.
  

<p align = "center">
   <img src = "/assets/images/SNS rule2.PNG"> <br/>
   Lambda 사용하지 않을 때 Rule 설정
</p>

<p align = "center">
   <img src = "/assets/images/SNS rule3.PNG"> <br/>
   json 파일 형식으로 오는 메일
</p>


# 튜토리얼

## Simple Notification Service 주제 생성

1. `Amazon SNS 콘솔`로 이동
2. `주제` → `주제 생성` 클릭
3. 아래 그림처럼 체크 후 `주제 생성` 클릭

<p align = "center">
   <img src = "/assets/images/SNS1.PNG"> <br/>
   표준, 이름 작성
</p>

4. 주제의 디테일 페이지에서 구독 생성
    - `구독 생성` 클릭
    - 프로토콜: email
    - 앤드포인트: 이메일을 받을 주소
    - `구독 생성` 클릭해서 구독 만들기
5. 메일함에서  `AWS Notification - Subscription Confirmation` 라는 메일을 받았다면 메일 내 링크를 클릭(`Confirm subscription`)해서 구독 확인하기  

<p align = "center">
   <img src = "/assets/images/SNS3.PNG"> <br/>
   구독 확인 메일
</p>

### Rule이나 Lambda를 사용하지 않고 메시지 직접 작성해서 메일로 보내기

- rule이나 Lambda를 사용하지 않고 바로 메시지를 작성해서 메일로 보내고 싶다면 위의 단계를 완료 후 이 부분을 수행해주세요.
- Rule, Lambda를 사용하고자 한다면 이 부분은 무시하고 진행해주세요.
1. Amazon SNS에서 아까 생성한 주제의 디테일에서 `메시지 게시` 선택
2. 엔드포인트로 전송할 메시지 본문에 원하는 메시지 내용 게시 후 `메시지 게시` 클릭

## Lambda 함수 생성

1. `AWS Lambda 콘솔`로 이동
2. `함수` → `함수 생성`을 클릭
    - `블루프린트 사용` 선택
    - `hello-world-python` 선택
    - `hello-world-python` 이름 클릭해서 상세 설정 페이지로 이동
    - 아래 사진처럼 선택 후 함수 생성 클릭

<p align = "center">
   <img src = "/assets/images/LAMBDA1.PNG"> <br/>
   Lambda 
</p>

3. 함수 디테일 페이지에서 `코드` → `lambda [function.py](http://function.py)` 아래와 같이 수정 후 deploy 클릭

    ```python
    import boto3
    #
    #   expects event parameter to contain:
    #   {
    #       "device_id": "32",
    #       "reported_temperature": 38,
    #       "max_temperature": 30,
    #       "notify_topic_arn": "arn:aws:sns:us-east-1:57EXAMPLE833:high_temp_notice"
    #   }
    # 
    #   sends a plain text string to be used in a text message
    #
    #      "Device {0} reports a temperature of {1}, which exceeds the limit of {2}."
    #   
    #   where:
    #       {0} is the device_id value
    #       {1} is the reported_temperature value
    #       {2} is the max_temperature value
    #
    def lambda_handler(event, context):

        # Create an SNS client to send notification
        sns = boto3.client('sns')

        # Format text message from data
        message_text = "Device {0} reports a temperature of {1}, which exceeds the limit of {2}.".format(
                str(event['device_id']),
                str(event['reported_temperature']),
                str(event['max_temperature'])
            )

        # Publish the formatted message
        response = sns.publish(
                TopicArn = event['notify_topic_arn'],
                Message = message_text
            )

        return response
    ```

4. SNS ARN 알아내기
    - Amazon SNS에서 만들어 놓은 주제에 들어가 ARN 확인
    - 다음 단계에서 사용되므로 복사해놓기
5. Lambda console에서 아까 만든 함수의 디테일 페이지로 이동
6. `코드` 의  윗부분에 `Test ▽` 에서 삼각형을 눌러 `Configure test event` 선택

<p align = "center">
   <img src = "/assets/images/LAMBDA3.PNG"> <br/>
   Lambda test 생성
</p>

   - `새로운 테스트 이벤트 생성` 선택
   - 이벤트 이름: SampleRuleOutput
   - 코드 부분에 존재하던 내용 모두 지우고 아래 내용 붙여넣고 `생성` 클릭


   ```json
    {
      "device_id": "32",
      "reported_temperature": 38,
      "max_temperature": 30,
      "notify_topic_arn": "arn:aws:sns:us-east-1:57EXAMPLE833:high_temp_notice"
    }  
   ```

## Rule 생성

1. `AWS IoT console` → `동작` → `규칙` →  `생성` 클릭
    - 이름: wx_friendly_test
    - SQL 버전: 2016-03-03
    - 규칙 쿼리 설명문

    ```sql
    SELECT 
      cast(topic(2) AS DECIMAL) as device_id, 
      temperature as reported_temperature,
      30 as max_temperature,
      'arn:aws:sns:us-east-1:57EXAMPLE833:high_temp_notice' as notify_topic_arn
    FROM 'device/+/data' WHERE temperature > 30
    ```

    에서 아까 알아낸 SNS의 arn을 대신 적어주세요. (''로 감싸는 거 잊지 마시구요)

    - 작업 구성 선택
        - `메시지 데이터를 전달하는 Lambda 함수 호출` 선택
        - 함수 이름: format-high-temp-notification(위에서 만든 lambda 함수 선택)
    - `규칙 생성` 클릭

## 테스트

1. `AWS IoT console` → `동작` → `테스트`  클릭
2. `주제 구독` 에서
    - 주제 필터: device/+/data
    - `구독` 클릭
3.  `주제 개시` 에서
    - 주제 이름: device/32/data
    - 메시지 페이로드에 아래와 같이 작성(사실 temperature만 있어도 특별히 상관 없습니다.)

    ```json
    {
      "temperature": 38,
      "humidity": 80,
      "barometer": 1013,
      "wind": {
        "velocity": 22,
        "bearing": 255
      }
    }
    ```

    - 위 Rule에서 30도 초과면 메일을 보내도록 설정해놨기 때문에 temperature는 30 이상인 값으로 작성해야 메일이 날아옵니다.
    - 반대로 30도 이하면 메일이 날아오지 않는지 확인하는 것도 가능합니다.
    - `게시` 를 클릭한 후 밑에서 device/+/data 에 작성한 내용이 보내졌는지 확인합니다.
    - 메일함에 다음과 같은 메일을 받았는지 확인합니다.

<p align = "center">
   <img src = "/assets/images/LAMBDA4.PNG"> <br/>
   받은 메일(Lambda 사용 이전과 달리 json 형식이 아님)
</p>

# 결론

Lambda 함수와 SNS 기능을 이용하여 특정 상황일 때 자동적으로 메일을 보내는 예제를 학습했습니다. Lambda 함수의 코드를 작성하는데 조금 더 능숙해져야 할 것 같습니다. 이 예제를 응용하면 디바이스 관리 및 모니터링이 훨씬 쉬워질 것으로 보입니다. 다음에는 비슷한 방식으로 Rule을 이용하여 dynamoDB 테이블에 값을 저장하는 예제를 진행할 예정입니다.

이 글이 도움이 되셨다면 댓글로 알려주세요^^
