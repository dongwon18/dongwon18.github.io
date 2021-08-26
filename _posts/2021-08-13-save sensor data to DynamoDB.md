---
title: "센서 데이터 실시간으로 DynamoDB에 저장하기" 
excerpt: "AWS IoT Rule 이용 RPI 측정 센서값 MQTT로 DynamoDB에 저장하기"
date: 2021-08-13
categories: 
  - SensorMonitoring
tags:
  - AWS_IoT
  - Tutorial
  - RPI
---

이 글은 [AWS IoT를 이용하여 센서 값 모니터링 하는 IoT 시스템 구축 프로젝트](https://dongwon18.github.io/categories/#sensormonitoring)의 네 번째 글입니다.

안녕하세요. 디바이스에서 받은 값을 단순히 csv 파일로만 저장한다면 search, sort 등 데이터를 다룰 때 추가적인 작업을 진행해야 합니다. 값을 클라우드 데이터베이스에 저장하면 search, sort 등이 쉬울 뿐 아니라 다른 시스템에서 접근하기도 쉽습니다. 확장성이 좋아진다고 볼 수 있죠. 따라서 디바이스에서 받은 값을 DynamoDB에 저장해볼 예정입니다.

전체 아키텍처로 보자면 파란색 부분입니다. 

<p align = "center">
  <img src= "/assets/images/프로젝트4.png" alt = "total architecture">
  전체 아키텍처 중 구현한 부분
</p>

# 목표

- 센서에서 읽은 값을 MQTT 프로토콜을 이용하여 지정한 주제에 게시한다.
- 해당 값을 DynamoDB에 저장한다.
- AWS IoT를 이용하여 구현한다.

# 사전 정보

기본적인 AWS IoT 사용법에 관련된 내용은 블로그 [AWS_IoT](https://dongwon18.github.io/categories/#aws_iot) 카테코리에 있으니 참고해주세요. 이번 실습과 연관된 글은 다음과 같습니다.

- [AWS IoT Rule 개념, 사용하기](https://dongwon18.github.io/aws_iot/AWS-IoT-Rule-tutorial/)
- [AWS IoT 이용 디바이스 값 DynamoDB에 저장하기](https://dongwon18.github.io/aws_iot/Store-to-DynamoDB-in-AWS-IoT/)
- [Python으로 RPI에서 센서 값 읽어 와 AWS IoT에 전송하기](https://dongwon18.github.io/sensormonitoring/send-sensor-value-from-RPI-to-AWS/)

# 사용한 AWS

- AWS IoT: Rule
- DynamoDB

# 전체적인 흐름

1. 데이터를 저장할 테이블을 DynamoDB에서 만듭니다.
2. 디바이스에서 측정한 값을 MQTT 프로토콜을 이용하여 지정된 주제에 게시합니다.
3. AWS IoT Rule을 이용하여 해당 주제로 전송된 데이터를 DynamoDB에 저장합니다.

# 튜토리얼

## DynamoDB 테이블

<div class="notice--info" markdown='1'>
**변경 사항**  
이후 부분을 진행하면서 필요에 의해 DynamoDB의 key를 변경하였습니다. 변경된 사항은 다음과 같습니다.

파티션 키: client_id(문자)  
정렬 키: store_time(숫자)
</div>


1. `AWS DynamoDB 콘솔 -> 테이블` 에서 테이블을 생성합니다.  
    - 테이블 이름: RPI_data
    - 파티션 키: store_time(숫자)
    - 정렬 키: client_id(문자)
    - 설정: 기본 설정

## AWS IoT Rule

1. `AWS IoT Console -> 동작 -> 규칙` 에서 규칙을 생성합니다.
    - 이름: RPI_data_store
    - 규칙 쿼리 설명문
```SQL
SELECT
  cast(temperature AS DECIMAL) as temperature, 
  humidity
FROM
  'device/+/data'
```
    - 작업: DynamoDB 테이블에 메시지 삽입하기
        - 테이블 이름: 이전에 만들었던 테이블을 선택
        - 파티션 키: ${timestamp()}
        - 범위 키: ${topic(2)}
        - 이 열에 메시지 데이터 쓰기: sensor_data(데이터 열 이름)
    - 새로운 역할 만들기(이름은 마음대로 지으셔도 됩니다.)

## 디바이스 통신 코드

코드가 길어서 [깃허브](https://github.com/dongwon18/AWS_IoT_SensorMonitoring/blob/main/send_sensor_by_MQTT.py)에 올려놓았습니다.

해당 코드를 실행하기 위헤서는 제가 이전 글에서 했던 것처럼 인증서의 위치와 이름을 변경해 놓아야 합니다.

파일이 있는 디렉터리로 이동 후

```python
python3 send_sensor_by_MQTT.py --endpoint your-endpoint --client-id RPI --topic device/RPI/data
```

로 파일을 실행해 주세요. `your-endpoint` 에는 본인의 엔드포인트를 대신 넣어야 합니다.

## 결과

- 결과는 `DynamoDB 콘솔 -> 항목 -> 테이블 선택` 을 통해 확인하실 수 있습니다.

<p align = "center">
  <img src= "/assets/images/store_value4.PNG" alt = "stored value in DynamoDB"> <br/>
  DynamoDB에 저장된 데이터
</p>


- 그림에서 보는 것 처럼 이름이 바로 표시되지 않고  `"N"` 으로 표시됩니다. 근데 MQTT 주제에는 또 제대로 표시 되더라고요.  
이런 식으로요.

```json
{
	"temperature" : 24,
	"humidity" : 43
} 
```

Rule 의 SQL에서 cast를 이용해서 형식을 decimal로도 바꿔봤지만 결과가 동일해서 일단 이대로 두고 있습니다. 

# 결론

- 실시간으로 센서에서 읽어온 값을 AWS IoT를 이용하여 DynamoDB에 저장하는 것에 성공했습니다.
- DynamoDB에 저장된 값의 형식에 문제가 있어 조치가 필요합니다.

# 참고

- AWS Developer Guide
