---
title: "디바이스 연결 상태 이메일 받기" 
excerpt: "AWS IoT, SNS, Lambda 이용 디바이스 연결 끊겼을 때 이메일 받기"
date: 2021-08-12
categories: 
  - SensorMonitoring
tags:
  - AWS_IoT
  - Tutorial
---

이 글은 [AWS IoT를 이용하여 센서 값 모니터링 하는 IoT 시스템 구축 프로젝트](https://dongwon18.github.io/categories/#sensormonitoring)의 세 번째 글입니다.

안녕하세요. 좋은 클라우드 IoT 환경을 구성하기 위해서는 디바이스들의 연결 상태를 상시 모니터링해야 하고 만약 문제가 생긴다면 즉각적으로 조치를 취해줘야 합니다. 이번에는 그 부분을 AWS IoT에서 구현해 볼 예정입니다. 

전체 아키텍처로 보자면 초록색 박스 부분입니다. 

<p align = "center">
  <img src = "/assets/images/프로젝트2.png" alt = "total architecture"> <br/>
  이 실습의 해당 부분
</p>

# 목표

- 시스템 예약 주제를 구독하여 디바이스 연결 상태 모니터링
- 만약 디바이스 연결이 끊겼다면 담당자에게 이메일 보내기
- AWS IoT Rule을 이용해 구현하기

# 사전 정보

기본적인 AWS IoT 사용법에 관련된 내용은 블로그 [AWS_IoT 카테고리](https://dongwon18.github.io/categories/#aws_iot)에 있으니 참고해주세요. 이번 실습과 연관된 글은 다음과 같습니다. 

- AWS IoT Rule 개념, 사용하기([블로그 링크](https://dongwon18.github.io/aws_iot/AWS-IoT-Rule-tutorial/))
- Rule 이용하여 특정 조건일 때 메일로 알리기([블로그 링크](https://dongwon18.github.io/aws_iot/Sending-email-in-AWS-IoT/))
- AWS IoT 디바이스 모니터링([블로그 링크](https://dongwon18.github.io/aws_iot/get-connection-state-using-LWT-in-AWS-IoT/))

# 사용한 AWS

- AWS IoT: Rule, MQTT 시스템 예약 주제
- AWS Lambda: 메일 메시지 형식 맞추기
- AWS SNS: 메일 보내기
- AWS IAM: Lambda 함수에서 역할 생성

# 전체적인 흐름

1. 라즈베리파이와 AWS IoT를 연결합니다.
2. 지정된 시스템 예약 주제로 연결 상태를 모니터링 합니다.
3. 만약 연결이 끊기면 AWS IoT에서 시스템 예약 주제인  `$aws/events/presence/disconnected/+`로 관련 정보를 게시합니다.
4. AWS IoT Rule을 이용하여 해당 주제에서 SQL을 이용하여 필요한 정보만 뽑아내고 Lambda 함수를 실행합니다.
5. Lambda 함수에서 메시지를 메일에서 봤을 때 깔끔하도록 모양을 다듬습니다.
6. Lambda 함수에서 메시지를 다 다듬었으면 AWS SNS를 실행합니다. 
7. AWS SNS에서는 Lambda 함수에서 게시한 내용을 이메일로 송신합니다. 

# 튜토리얼

이전 글을 따라하면서 저랑 같은 세팅이 되어 있다고 가정하고 진행하겠습니다.

## SNS 주제 만들기 및 구독

1. `Amazon SNS  콘솔 -> 주제` 에서 주제를 생성합니다.
    - 표준 선택
    - 이름: disconnected_notice
2. 생성된 주제에서 `구독 생성` 을 합니다.
    - 프로토콜: 이메일
    - 엔드포인트: disconnected 메시지를 받을 이메일 주소

## Lambda 함수 생성

1. `AWS Lambda 콘솔 -> 함수` 에서 함수를 생성합니다.
    - 블루프린트 사용
    - hello-world-python 찾기
        - `hello-world-python` 클릭해서 세부 설정 페이지로 이동
        - 함수 이름:  format-disconnected-notification
        - `AWS 정책 템플릿에서 새 역할 생성` 선택
            - 새 역할 이름: format-disconnected-notification-role
        - 함수 생성 선택
2. 함수 상세 페이지로 이동
    - 코드에서 아래와 같이 입력 후 `Deploy`

<script src="[https://gist.github.com/dongwon18/882e3433a473a2821afeef2404015c7a.js](https://gist.github.com/dongwon18/882e3433a473a2821afeef2404015c7a.js)"></script>

3. 테스트 하기
    - 코드 페이지의 <span style="background-color:orange;">Test ▼ </span>클릭 → configure test event 선택
    - 새로운 테스트 이벤트 생성
        - 이벤트 템플릿: Sample
        - 이벤트 이름: (아무거나)
        - 아래와 같이 입력 후 저장
<script src="[https://gist.github.com/dongwon18/4ea29d35af94aefbe0d6971b93725e07.js](https://gist.github.com/dongwon18/4ea29d35af94aefbe0d6971b93725e07.js)"></script> <br/>
   SNS arn은 아까 만든 SNS 주제의 arn 을 넣어주세요.
   - `Test` 클릭하여 결과 확인  
    (error 가 안 나오면 됩니다.)

## Rule 만들기

1. `AWS IoT 콘솔 -> 규칙` 에서 새로운 규칙을 만듭니다.
    - 이름: disconnection_emial_send
    - 규칙 쿼리 설명문

<script src="[https://gist.github.com/dongwon18/50b90153ec56ce2de5bd35e64315b952.js](https://gist.github.com/dongwon18/50b90153ec56ce2de5bd35e64315b952.js)"></script> <br/>   
   이렇게 입력하면 `clientInitiatedDisconnect` 가 true 여도 메일을 보냅니다. 사용자가 `ctrl + c` 를 입력해서 연결이 끊기거나 프로그램이 끝나서 연결이 끊기는 경우가 이에 해당합니다. 예상치 못한 연결 끊김이 아닐 때도 포함하는 거죠.  
   테스트 등을 위해서 사용자가 연결을 끊을 때 항상 관리자에게 메일을 보낸다면 매우 귀찮고 어떤 메일이 진짜 끊겨서 온 것인지 알기 어렵겠죠? 그래서 아래와 같이 코드를 작성하면 예상치 못한 연결 끊김일 때만 메일을 송신합니다. 다만 저는 이 rule을 테스트 하기 쉽게 위의 코드대로 진행하겠습니다.
<script src="[https://gist.github.com/dongwon18/66fa8d4e18caaf6597bf4d4cf0794f87.js](https://gist.github.com/dongwon18/66fa8d4e18caaf6597bf4d4cf0794f87.js)"></script>

2.  작업으로는 `메시지 데이터를 전달하는 Lambda 함수 호출` 선택해서 이전에 만들었던 Lambda 함수를 선택해줍니다.(저는 format-disconnected-notification) 

<p align = "center">
  <img src = "/assets/images/noice_disconnection4.PNG" alt = "create rule"> <br/>
  만들어진 Rule 모습
</p>
 
3. 규칙을 생성하고 규칙을 활성화했는지 확인합니다.

## 결과

- `AWS IoT 콘솔 -> 테스트` 에서 결과 확인을 위해 `$aws/events/presence/connected/+` 와 `$aws/events/presence/disconnected/+` 를 구독합니다.   
가끔 connection closed 등의 에러로 구독이 안 될 때도 있는데 그럴 때는 그냥 새로 고침하면 구독이 잘 됩니다.
- 이전 글에서 connected는 메시지를 못 받았다고 했었는데 이번에는 또 잘 받네요. 아래와 같이 해당 주제로 메시지를 잘 받고 있음을 확인합니다.

<p align = "center">
  <img src = "/assets/images/noice_disconnection2.PNG" alt = "subscribe connected topic"> <br/>
  시스템 예약 주제 구독: Connection 정보 
</p>

<p align = "center">
  <img src = "/assets/images/noice_disconnection3.PNG" alt = "subscribe disconnected topic"> <br/>
  시스템 예약 주제 구독: Disconnection 정보
</p>


- 아래와 같이 연결이 끊겼을 때 메일을 정상적으로 받았습니다.

<p align = "center">
  <img src = "/assets/images/noice_disconnection.PNG" alt = "get notice email"> <br/>
  Notification email 수신
</p>


# 결론

- Amazon 서비스들을 이용해서 디바이스의 연결이 끊겼을 때 관리자에게 이메일을 보내는 부분을 완성했습니다.

## 개선 사항

- 실제로 사용할 때는 `clientInitiatedDisconnected` 가 false일 때만 메일을 보내서 관리자가 연결이 실제로 디바이스에 문제가 있을 때만 메일을 수신하도록 할 것

# 참고

- AWS IoT Development Guide
- [Timestamp를 time으로 변경하고 시간대 바꾸기](https://jwkim96.tistory.com/145)
