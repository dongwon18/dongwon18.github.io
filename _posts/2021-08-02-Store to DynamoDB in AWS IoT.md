---
title: "AWS IoT 이용 디바이스 값 DynamoDB에 저장하기"
excerpt: "AWS IoT, DynamoDB 연결"
date: 2021-08-02
categories:
  - Tutorial
  - AWS_IoT
---

안녕하세요. 이번에는 MQTT 프로토콜을 이용해서 디바이스로부터 받은 값을 AWS IoT Rule을 이용해DynamoDB에 저장하는 예제를 진행해보겠습니다.

# 사용한 AWS 서비스

- AWS IoT: Rule 사용
    - 계정 생성 후 12개월 동안 제공되는 프리 티어는 다음과 같습니다.
        - 연결 2,250,000분
        - 메시지 500,000개
        - 레지스트리 또는 디바이스 섀도 작업 225,000건
        - 트리거된 규칙 250,000건 및 실행된 작업 250,000건
- DynamoDB
    - 데이터 스토리지 25GB
    - DynamoDB Streams로부터 250만 건의 스트림 읽기 요청
    - 모든 AWS 서비스 합계 데이터 전송량 1GB
- 이번 예제를 끝내고 나니 다음과 같이 표시되더라고요.
    - 프로비저닝된 읽기 용량: 5
    - 프로비저닝된 쓰기 용량: 5
- 예제를 통해 한 게 정확히 어떤 작업인지 알기 어려워서 프리 티어에 해당하는지 잘 모르겠습니다.

# 전체적인 흐름
1. DynamoDB에 센서 값을 저장할 테이블을 만든다.
2. AWS IoT에서 MQTT 프로토콜을 통해 디바이스에서 값을 받아온다.
3. AWS IoT Rule의 SQL을 이용하여 데이터를 수정 후 DynamoDB에 값을 저장한다.  

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

이렇게 wind로 인해 2차원으로 묶여있던 값을 SQL을 이용하여 1차원으로 펴줄 겁니다. 요렇게요.

```json
{
  "temperature": 28,
  "humidity": 80,
  "barometer": 1013,
  "wind_velocity": 22,
  "wind_bearing": 255
}
```

# 튜토리얼

해당 튜토리얼에서는 직접 디바이스에서 값을 받아오는 대신 테스트 기능을 이용하여 진행했습니다.

## DynamoDB 테이블 생성

1. `AWS DynamoDB console` → `테이블` → `테이블 만들기`로 이동
2. 아래 사진 처럼 선택

<p align = "center">
  <img src = "/assets/images/dynamoDB1.PNG"> <br/>
  테이블 생성
</p>

  - primary key는 디바이스에서 값이 측정되었을 때의 시간을 timestamp로 나타낸 값
    primary key는 중복될 수 없는 row마다 고유한 값이므로 시간으로 설정하는 것이 바람직하다.(한 번에 여러 디바이스에서 값이 올라오는 경우는 없다고 가정하는 듯 합니다.)
  - sort key는 디바이스 번호

## 규칙(Rule) 만들기

1. `AWS IoT console` → `동작` → `규칙` → `생성` 클릭
    - 이름: wx_data_dbb
    - 규칙 쿼리 설명문

    ```sql
    SELECT temperature, humidity, barometer,
      wind.velocity as wind_velocity,
      wind.bearing as wind_bearing,
    FROM 'device/+/data'
    ```
    - `wind.velocity as wind_velocity` 를 통해 wind 내부에 있던 값을 1차원으로 끌어내주는 겁니다.    


2. 하나 이상의 작업을 설정에서 `작업 추가` 선택
    - `DynamoDB 테이블에 메시지 삽입하기` 선택 후 `작업 구성` 클릭
    - 아래 사진과 같이 설정(사진에 보이는 것처럼 역할도 새로 생성)

<p align = "center">
  <img src = "/assets/images/DB1.PNG"> <br/>
  DynamoDB 테이블에 메시지 삽입하기 설정
</p>


<p align = "center">
  <img src = "/assets/images/dynamoDB3.PNG"> <br/>
  규칙 생성
</p>

3. 규칙 생성

## 테스트

이 부분은 이전 포스팅에서 자세히 다뤄서 이번에는 간략하게만 다뤘으니 헷갈리신다면 [이전 포스팅](https://dongwon18.github.io/tutorial/aws_iot/AWS-IoT-Rule-tutorial/)을 참고해주세요.
1. `AWS IoT console` → `동작` → `테스트` 선택
2. `device/+/data` 구독
3. `device/22/data` 에 아래 내용 작성 및 게시

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

1. `DynamoDB console` → `테이블` → 아까 만든 테이블로 이동
2. `항목` 에서 추가된 값 확인

<p align = "center">
  <img src = "/assets/images/dynamoDB4.PNG"> <br/>
  테이블에 생성된 값 확인
</p>

# 결론

Rule을 이용하여 디바이스로부터 전달된 값을 수정하여 DynamoDB에 저장하는 예제를 학습했습니다. Rule을 이용하면 DynamoDB 뿐 아니라 18개의 다른 Amazon 서비스로 연결 가능하기 때문에 활용하면 좋을 듯 합니다.

# 참고

- [AWS IoT 자습서](https://docs.aws.amazon.com/iot/latest/developerguide/iot-ddb-rule.html)

이 글이 도움이 되셨다면 댓글로 알려주세요.^^ 

오류가 있을 시 언제든지 댓글을 통해 알려주세요.
