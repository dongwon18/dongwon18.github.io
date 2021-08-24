---
title: "Node Red 이용하여 MQTT 주제 구독하기"
excerpt: "Node Red, AWS IoT MQTT 이용 센서 값 브라우저에서 확인하기"
date:2021-08-24
categories:
  - IoT
tags:
  - Tutorial
  - AWS_IoT
  - Node_RED
  - RPI
---

안녕하세요. 이전 글에서 센서 값을 실시간으로 확인하는 방법 중 Node-RED를 이용하여 MQTT 주제 자체를 구독하여 확인하는 방법도 가능하다고 언급한 적이 있습니다. 이 글에서는 그 부분을 다뤄볼 예정입니다.

# 목표

- 라즈베리파이에 연결되어 있는 센서에서 오는 데이터를 실시간으로 확인할 수 있도록 한다.
- MQTT 통신 중 라즈베리파이가 데이터를 보내는 주제를 Node-RED에서 구독하는 방식으로 값을 가져온다.

# 사전 정보

- [Python으로 RPI에서 센서 값 읽어 와 AWS IoT에 전송하기](https://dongwon18.github.io/sensormonitoring/send-sensor-value-from-RPI-to-AWS/)

    이 실습에서 사용한 MQTT 프로토콜은 AWS IoT에서 제공하는 MQTT를 사용하였습니다. 이전 글에서 센서 데이터를 MQTT 프로토콜을 이용하여 AWS IoT에 보내는 코드를 작성했었는데 그 코드를 그대로 사용할 예정입니다.    

# 사용한 것

- 라즈베리파이
- AWS IoT MQTT 서비스, 인증
- Node RED(노드 레드)

# 전체적인 흐름

1. 라즈베리파이에서 센서 값을 읽어온다.
2. 읽어온 값을 지정된 MQTT 주제에 게시한다.
3. Node-RED에서 해당 주제를 구독하여 라즈베리파이에서 보낸 값을 확인한다.

# 튜토리얼

1. [이전 글](https://dongwon18.github.io/aws_iot/start-AWS-IoT/#aws-iot%EC%97%90-%EB%94%94%EB%B0%94%EC%9D%B4%EC%8A%A4-%EC%97%B0%EA%B2%B0)을 참고하여 AWS IoT에서 사물을 생성하고 인증서를 설치합니다.
2. 라즈베리파이에 Node-RED를 설치합니다.   
2015년 이후 OS를 받은 분들은 이미 설치가 되어 있다고 하는데 저는 제일 최신 OS를 2021년에 다운 받았음에도 설치가 되어 있지 않아 따로 설치하였습니다. cmd에 `node-red` 라고 입력했을 때 에러가 뜨면 설치가 안 되어 있는 것이니 설치를 해주시기 바랍니다.
3. cmd에 `node-red` 를 입력하여 실행합니다.   
실행 후 브라우저 창이 바로 뜰 때도 있는데 안 뜬다면 cmd에 출력된 `내부 ip:1880` 형태의 주소를 브라우저에 직접 쳐서 node-red 화면을 띄워도 됩니다. (`localhost:1880` 도 가능)
4. 아래와 같이 MQTT 주제를 구독하고 값을 받아옵니다.(연필 모양 클릭으로 설정 화면 진입)

<p align = "center">
  <img src = "/assets/images/NODE_RED.PNG" alt = "Node Red coding"> <br/>
  Node Red 코딩
</p>

    - 서버: AWS IoT endpoint 입력
    - 포트: 8883
    - `사용 TLS` 활성
        - TLS 설정 옆 연필 모양 클릭
            - 이전에 다운로드 받아 놓은 인증서 3개를 각각 등록
            - 저장하기
    - QoS: 0
    - Protocol: MQTT v3.1.1   
    AWS IoT의 MQTT는 실제로는 v3.1.1과는 달리 QoS를 0, 1만 지원합니다. (저는 처음에 Node-RED QoS 기본이 2로 설정되어 있는데 상관 없을 거라고 생각하고 진행하고 연결이 안 되었는데 그 이유가 이 때문이었습니다.) 그 외에도 와일드 카드 사용 시 parent 주제 구독 여부 등 차이가 있으니 자세한 건 [AWS IoT Developer Guide의 내용](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html#mqtt-differences)을 참고해주세요.
    - 주제: device/RPI/data   
    이전에 제가 작성한 코드에서 데이터를 게시하는 주제입니다.
    - 저장하기
5. 라즈베리파이에서 이전 글에서 작성했던 send_sensor_by_MQTT.py를 실행하여 센서에서 값을 읽어와 주제에 게시합니다.
6. Node-RED에서 디버그 창을 통해 메시지를 확인합니다.

## 결과

- 위 그림처럼 DynamoDB 콘솔에 들어가지 않아도 값을 확인할 수 있는 것을 볼 수 있습니다.
- 이 방식대로만 하면 같은 네트워크에 연결되어 있어야만 값을 확인할 수 있습니다. 즉 라즈베리파이와 같은 네트워크를 사용하지 못할 만큼 거리가 멀어진다면 이 방식으로는 센서의 값을 확인할 수 없다는 뜻입니다.

# 결론

- Node-RED를 이용하여 간단하게 MQTT 주제를 구독하여 센서 값을 확인할 수 있습니다.
- Node-RED에서는 게이지 등 다양한 GUI를 간단하게 사용할 수 있습니다. 따라서 필요에 따라 데이터를 손쉽게 시각화 하기 좋을 듯 합니다.
- Node-RED 앱을 스마트폰에 설치(play store에 존재)할 수 있습니다. 즉 앱을 굳이 제작하지 않아도 비교적 쉽게 값을 모니터링할 수 있습니다.
- 포트포워딩, DDNS 설정 등을 통해 같은 네트워크 뿐 아니라 외부 네트워크에서도 해당 Node-RED 페이지에 접속하여 값을 확인할 수 있습니다. 대신 추가적인 보안 작업이 필요합니다. password를 설정함으로써 보안을 강화할 수 있습니다.
- DynamoDB에 저장된 값을 실시간으로 불어오는 경우에 비해 delay가 적은 데이터를 확인할 수 있습니다. 그러나 Amazon 서비스들로 모든 부분을 제어한다는 프로젝트의 통일성 측면에서는 AWS Amplify를 이용하는 방식보단 떨어져 보입니다.

# 참고

- [두원공과대학교 김동일 교수님 Node-RED MQTT 통신](https://youtu.be/Imj9S7ierOU)    
Node-RED 안드로이드 앱을 이용하여 MQTT 주제를 구독하고 게시할 수 있는 방법을 확인할 수 있습니다.
- [Node-RED password 설정](https://m.blog.naver.com/seodaewoo/221220204466)    
라즈베리파이에서 Node-RED를 시작하고 password를 설정하는 방법을 확인할 수 있습니다.
- [AWS IoT MQTT Developer Guide](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html#mqtt-differences)   
MQTT 프로토콜의 특징과 사용법 등을 확인할 수 있습니다.
