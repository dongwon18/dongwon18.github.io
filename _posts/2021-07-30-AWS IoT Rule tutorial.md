---
title: AWS IoT Rule(Rule 개념, Rule 만들기, republish to MQTT 예제)
date: 2021-07-30
categories:
  - AWS_IoT
tags:
  - Tutorial
---
안녕하세요. 이전 글(AWS 사물 생성, 기기 등록 및 MQTT 통신 확인)에 이어 AWS IoT Rule에 대해 다뤄보겠습니다.  

# Rule 이란?

MQTT topic stream으로부터 SQL을 사용하여 필요한 데이터를 선택해 분석하고 명령을 내리는 기능입니다. AWS IoT 홈페이지에 따르면 Rule을 통해 할 수 있는 일은 다음과 같습니다.

- 디바이스로부터 받은 데이터 처리
- 디바이스에서 받은 데이터 DynamoDB나 S3에 저장
- Amazon SNS를 사용하는 사용자에게 푸시 알림 전송
- Amazon SQS 큐에 데이터 게시
- 데이터 추출을 위한 Lambda 함수 호출
- MQTT 메시지로부터 Amazon Machine Learning에 데이터를 보내 prediction 진행
- 웹 어플리케이션, 서비스 에게 메시지 데이터 전송

등 이 외에도 다양한 AWS의 서비스와의 연결이 가능합니다.

MQTT는 메시지를 게시(publish)하면 구독(subscribe)하는 형식의 프토토콜로 특정 주제를 구독하는 형식입니다. 디바이스에서 게시한 데이터는 AWS IoT 메시지 브로커가 주제에 따라 확인합니다. 데이터를 받고자 하는 디바이스는 특정 주제를 구독하고 토픽 필터를 브로커에게 보냄으로써 값을 받아올 수 있습니다. 

## AWS IoT Rules

AWS IoT Rule은 두 가지로 이뤄져 있습니다. `Rule query statement와 Rule action` 입니다. 

- Rule Query statement
보통 생각하는 SQL과 형식이 비슷하며 메시지로 받은 데이터를 명시된 SQL Query에 따라 처리합니다. 처리된 데이터는 Rule의 action에서 보내지는 데이터입니다. 즉 받은 데이터를 어떻게 가공하는지에 대한 부분으로 SQL문으로 작성해야 합니다.
- Rule Actioin
Query를 통해 처리된 데이터를 어떻게 할 것인지에 대한 부분입니다. 해당 예제에서는 다시 MQTT로 보내는 action을 하는데 이 외에도 앞에서 언급한 것처럼 DynamoDB나 S3에 저장하는 등 다양한 action을 할 수 있습니다.

# MQTT 통신하는 AWS IoT Rule 만들기

## Rule 생성

1. `AWS IoT console의 동작 -> 규칙` 으로 들어갑니다.
2. `생성` 을 클릭하여 새로운 rule을 만든다.
    1. 이름: republish_temp(마음대로 하셔도 상관 없습니다. 다만 해당 Region에서 겹치는 이름이 없어야 합니다.)
    2. 규칙 쿼리 설명문
        1. SQL 버전 사용: 2016-03-23
        2. 규칙 쿼리 설명문
        적혀 있던 내용을 지우고 아래 내용만 적습니다.

        ```sql
        SELECT topic(2) as device_id, temperature FROM 'device/+/data'
        ```

    - topic의 두 번째 즉 `device/+/data` 에서 + 에 해당하는 디바이스 번호를 `device_id` 라는 이름으로 갖고 오고 `device/+/data` 에 들어있는 `temperature` 에 해당하는 값을 가져오라는 뜻입니다.
    - '+' 는 와일드 카드로 어떤 문자열이 와도 상관없다는 뜻입니다. 이렇게 설정하면 다양한 디바이스 번호의 값을 받아올 수 있겠죠?
    - 결과적으로는 `device/22/data` 라는 토픽에서

    ```json
    {
      "temperature": 28,
      "humidity": 80,
      "barometer": 1013,
      "wind": {
        "velocity": 22,
        "bearing": 255
      }
    }
    ```

    를 데이터로 받았다면 

    ```json
    {
      "device_id": "22",
      "temperature": 28
    }
    ```

    가 Query의 결과입니다. 이 때 `device_id` 의 값이 ""로 둘러쌓인 string인 것을 알 수 있는데 temperature처럼 숫자로 변경하고 싶다면 규칙 쿼리 설명문을 아래처럼 변경하면 됩니다.

    ```sql
    SELECT cast(topic(2) as DECIMAL) as device_id, temperature FROM 'device/+/data'
    ```

      3.  하나 이상의 작업을 설정

    1. `작업 추가` 선택: `메시지를 AWS IoT 주제에 재게시` 선택 → `작업 구성` 선택
    - 주제: device/data/temp
    - 서비스 품질: 0 - 메시지가 0번 이상 전달됩니다.
    - 역할을 선택 혹은 생성
        - `역할 만들기` 선택
        - 이름: republish_role → `역할 생성` 선택
        - 방금 만든 역할 선택

    d. 계속 다음을 눌러 규칙 생성 완료

## 테스트

이번 테스트에는 라즈베리파이를 사용하지 않고 튜토리얼에서 알려주는 대로 MQTT client를 사용했습니다. MQTT client를 사용하면 디바이스 콘솔 출력값을 AWS IoT console에서 확인할 수 있습니다.

1. `AWS IoT console -> 동작 -> 테스트` 로 이동합니다.
2. `주제 구독 -> 주제 필터` 에 `device/+/data` 와 `device/data/temp` 를 각각 구독합니다.
3. `주제 게시` 로 이동
    - 주제 이름: device/22/data
    - 메시지 페이로드:

        ```json
        {
          "temperature": 28,
          "humidity": 80,
          "barometer": 1013,
          "wind": {
            "velocity": 22,
            "bearing": 255
          }
        }
        ```

    - `게시` 선택
4. `주제 구독` 으로 이동하여 `device/+/data` 탭에서 보낸 데이터가 그대로 왔는지 확인합니다.
`device/data/temp` 에 rule에서 정한 대로 device_id와 temperature가 수신되었는지 확인합니다.

## 결과

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127589566-c7c9970c-c48d-4674-8792-d1cb47942c48.png" > <br/>
  device/+/data 에서 확인 가능한 내용
</p> 

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127589664-38076cf8-692f-4cfe-9790-619a8d2a4cc1.png" > <br/>
  device/data/temp 에서 확인 가능한 내용
</p> 

위와 같이 잘 동작합니다. 

- 이 때 필터 이름을 잘못 작성하면 아무 일도 일어나지 않으니 혹시 변화가 없다면 이름을 잘못 작성하지 않았는지 한번 더 확인해보세요.

# 결론

간단한 규칙을 통해 원하는 데이터만 추출하는 방식을 익혔습니다. 이 예제를 응용한다면 디바이스에서 받은 데이터 중 원하는 값만 SQL문을 통해 추출하여 원하는 AWS 서비스와 action을 통해 연결할 수 있습니다. 

AWS IoT를 이용하여 라즈베리파이에서 값을 받아 DynamoDB에 저장하고 측정한 센서 값을 머신 러닝으로 돌려 오류 예측 등을 하는 테스트베드 구축을 목표로 하고 있습니다. 이 때 rule을 이용하면 원하는 결과를 얻을 수 있을 것으로 보입니다. 

이 글이 도움이 되셨다면 댓글로 알려주세요^^
