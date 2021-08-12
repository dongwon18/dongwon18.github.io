---
title: "Python으로 RPI에서 센서 값 읽어 와 AWS IoT에 전송하기"
excerpt: "Raspberrypi와 AWS IoT Python 이용해 MQTT 통신하기"
date: 2021-08-10
categories:
  - SensorMonitoring
tags:
  - AWS_IoT
  - Tutorial
  - Python
---

안녕하세요. 프로젝트 첫 번째 단계로 센서에서 값을 읽어와  AWS IoT에 MQTT 프로토콜을 이용하여 데이터를 전송해보겠습니다. 사용된 코드는 AWS IoT의 예제 코드를 참고하였습니다.  
전체 아키텍처 상으로 확인해보면 노란색 부분입니다. 


<p align = 'center'>
  <img src = "/assets/images/프로젝트1.png"> <br/>
  이번 실습에서 진행할 부분
</p>

# 목표

- DHT22 센서에서 읽은 온도, 습도 값을 라즈베리파이에 전송한다.
- 라즈베리파이에서 AWS IoT에 데이터를 전송한다.

# 사용한 것

- 라즈베리파이 4B(Raspberrypi OS 32 bit, 이전 튜토리얼과 같이 인증서, SDK가 설치되어 있는)
- DHT22 센서, 빵판 및 점프선
- AWS IoT

# 전체적인 흐름

1. DHT22 센서에서 RPI로 측정한 온도, 습도 데이터를 디지털 신호로 전송한다.
2. 전송 받은 값을 RPI에서 json 파일로 변환한다.
3. 변환한 json 파일을 AWS IoT로 MQTT 프로토콜을 이용하여 전송한다.
4. AWS IoT에서 해당 주제를 구독하여 값을 확인한다.

# 진행 내용

1. 센서와 라즈베리파이를 하드웨어적으로 연결한다.

<p align = "center">
  <img src = "/assets/images/RPI GPIO.jpg" alt = "RPI & sensor connection"> <br/>
  라즈베리파이, 센서 연결  
</p>


|GPIO 번호|기능|
|:---:|:---:|
|2|5V 전압|
|7|GPIO 4(Digital input)|
|9|GND|




2. 라즈베리파이에 DHT22 센서를 사용하기 위한 라이브러리를 설치한다.

    ```arduino
    sudo apt-get update
    sudo apt-get upgrade
    sudo pip3 install Adafruit_DHT
    ```

3. 센서값을 읽어오기 위한 코드를 작성한다.

    ```python
    # file name: get_dht.py
    import Adafruit_DHT

    # define sensor and pin
    DHT_SENSOR = Adafruit_DHT.DHT22
    DHT_PIN = 4 # GPIO4
    POS = "TH01"

    # read sensor value
    t, h = Adafruit_DHT.read_retry(DHT_SENSOR, DHT_PIN)
    if(h == None or t == None):
        print("Reading failed")
    else:
        print("{}C {}%".format(h, t))
    ```

4. 읽어온 값을  json 파일로 변환한다.

    ```python
    payload = {"temperature": t, "humidity": h}
    payload = json.dumps(payload)
    ```

5. AWS IoT에 MQTT 프로토콜로 데이터를 전송한다.

    해당 부분은 코드가 길어서 [깃허브](https://github.com/dongwon18/AWS_IoT_SensorMonitoring/blob/main/send_sensor_by_MQTT.py)에 게시  
    기본적인 틀은 AWS IoT 예시 코드 참조  
    테스트를 위해 5번만 메시지를 송신하도록 설정

6. AWS IoT 테스트에서 확인한다.  

## 결과

- 코드 실행하기

```
cd ~/aws-iot-device-sdk-python-v2/samples
 python3 send_sensor_by_MQTT.py --endpoint your-end-point --thingName your-thing-name --client-id your-client-id --topic my/things/RPI/updat
```

- endpoint, 사물 이름, 클라이언트 이름, 토픽 이름을 지정해주세요.

<p align = 'center'>
  <img src = "/assets/images/SENSOR_SEND2.PNG"> <br/>
  5번 전송한 모습
</p>

<p align = 'center'>
  <img src = "/assets/images/SENSOR_SEND3.PNG"> <br/>
  AWS IoT Console test에서 해당 주제에 게시된 것 확인
</p>

<p align = 'center'>
  <img src = "/assets/images/SENSOR_SEND5.PNG"> <br/>
  ctrl + c 로 연결 끊기
</p>

위와 같이 측정한 값을 지정한 MQTT 주제로 게시하는 것을 확인할 수 있습니다.

# 결론

- 비록 중간에 아두이노를 태워 먹어서 센서 종류를 한 종류 밖에 쓸 수 없지만 기본적인 AWS IoT에 값을 보내는 단계까지는 완료했다.

## 개선 사항

- 실제로 사용할 때는 무한 루프를 돌려 계속 센서 값 측정하도록 설정
- 실제 테스트할 때는 적절한 측정 주기 설정
- 가능하다면 가지고 있는 센서를 추가하여 테스트 해볼 것

# 참고

- AWS IoT Development Guide
- [AWS IoT 카테고리의 글](http://dongwon.github.io/categories/#aws-iot)
