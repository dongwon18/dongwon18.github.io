---
title: "Sensor Monitoring 프로젝트 전체 아키텍처"
excerpt: "AWS IoT, Raspberry Pi를 이용한 Sensor Monitoring 전체 구조"
date: 2021-08-10
categories:
  - SensorMonitoring
tags:
  - AWS_IoT
  - Tutorial
  - RPI
---

안녕하세요. 지금까지 AWS IoT의 기본적인 사용법을 알아봤습니다. 이제 알아본 내용을 이용하여 프로젝트를 진행하려고 합니다. 먼저 프로젝트의 전체 아키텍처를 정리하고 실제 프로젝트를 진행하려 합니다.

# 프로젝트 목표

- 센서의 값을 안드로이드 어플리케이션으로 모니터링하는 시스템의 test bed 구축
- AWS IoT 사용

# 구현 목표

- 센서로 값을 측정한다.
- 측정한 값을 실시간으로 클라우드(DynamoDB)에 저장한다.
- 측정한 값은 실시간으로 안드로이드 어플리케이션으로 확인하도록 한다.
- 만약 측정 값이 위험 수위에 도달했다면 관리자에게 메일을 보내 적절한 조치를 취하도록 한다.
- 만약 디바이스 중 연결이 끊긴 디바이스가 있다면 관리자에게 메일을 보내 적절한 조치를 취하도록 한다.

# 프로젝트 아키텍처

<p align = "center">
  <img src = "/assets/images/AWS IoT Architecture (3).png" alt = "total architecture"> <br/>
  전체 아키텍처
</p>

프로젝트의 전체 아키텍처를 그려봤습니다. 
cf) 관련 icon을 찾아서 하나씩 다운 받다가 [AWS 홈페이지](https://aws.amazon.com/ko/architecture/icons/)에 있지 않을까? 하고 찾아보니까 역시 있더라고요. 다양한 프로그램에서 사용할 수 있도록 해놨는데 저는 draw.io를 사용했습니다. 예전부터 flow chart 그릴 때 사용했는데 간편하게 되더라고요.

## Sensor

IoT 현장에서 가장 많이 사용될 만한 센서를 골라봤습니다. 온습도 센서와 가스 감지 센서를 선택했습니다. 센서 테스트할 때는 구매처에서 제공한 코드를 참조했습니다.

- 온습도 센서(DHT22)(구매처 [링크](https://www.eduino.kr/product/detail.html?product_no=285&cate_no=34&display_group=1), 라이브러리 파일 제공)
  - DHT11 보다 정밀도가 높은 센서입니다. 
  - Digital data를 송신합니다. 
  - 단위: ºC, %
  - library 설치가 필요합니다.

```arduino
/*
 * test code of DHT22
 */
#include "DHT.h"

#define DHTPIN 2
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  dht.begin();

}

void loop() {
  // put your main code here, to run repeatedly:
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if(isnan(t) || isnan(h)){
    Serial.println("failed to read from DHT");
  }else{
    Serial.print("Humidity: ");
    Serial.print(h);
    Serial.print(" %\t");
    Serial.print("Temp: ");
    Serial.print(t);
    Serial.print(" *C\n");
  }
  delay(2000); 

}
```

- 알코올 센서(MQ-3)(구매처 [링크](https://www.eduino.kr/product/detail.html?product_no=275&cate_no=34&display_group=1&page_6=2#use_qna))
  - 다양한 가스 센서 중 테스트하기 가장 쉬울 것 같은 센서여서 골랐습니다.(주위에 손 소독제가 많으므로)
  - Analog data를 송신합니다.
  - 사용하는데 가열이 필요합니다.(3~ 5분)
  - 단위: ppm

```arduino
/*
 * test code of MQ-3
 */
int mq3Pin = A5;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  
}

void loop() {
  // put your main code here, to run repeatedly:
  Serial.println(analogRead(mq3Pin));
  delay(100);
  // int val = analogRead(mq3Pin);
}
```

## Arduino

아두이노 자체를 AWS IoT에 연결할 수도 있습니다. 저는 아두이노에서는 센서만 담당하고 실제적인 통신은 AWS IoT에서 담당할 수 있도록 프로젝트를 구성했습니다. 이렇게 하면 나중에 라즈베리파이에서 머신러닝이나 딥러낭을 이용한 예측 등 다른 기능을 추가하기 쉬울 것 같았기 때문입니다. 또한 아날로그 신호를 다루기 쉽습니다.

- Arduino Uno R3 SMD CH340 호환보드(구매처 [링크](https://www.eduino.kr/product/detail.html?product_no=59&cate_no=24&display_group=2&page_4=28), 호환보드 드라이버 제공)
- 기본적인 아두이노 보드입니다.  
(참고로 호환보드와 정품보드의 차이는 특별히 없다고 합니다. 아두이노 자체가 오픈 소스이기 때문에 보드를 제작한 회사의 차이라고 하더라고요. 후기를 참고 했을 때도 잘 동작한다고 하여 호환보드를 구매했습니다.)
- 호환보드 드라이버를 설치해야 합니다.  
(설치를 안 하면 USB로 연결했을 때 인식이 안 됩니다.)
    - 제공하는 드라이버 중 라즈베리파이 OS를 위한 드라이버는 없더라고요. 근데 드라이버 설치 없이 라즈베리파이랑 연결해서 사용해봤을 때 잘 동작했습니다.  
- 아두이노 우노는 디지털 신호와 아날로그 신호 모두 다룰 수 있기 때문에 위의 센서 두 개를 모두 사용할 수 있습니다.

## RaspberryPi

라즈베리파이는 사실 작은 컴퓨터 그 자체라고 보시면 됩니다. 모니터, 키보드, 마우스를 연결하면 그대로 컴퓨터로 사용할 수 있습니다. AWS IoT 자습서에서 라즈베리파이를 위한 방식도 제공하기 떄문에 처음에 접근하기 쉬울 뿐 아니라 프로젝트의 확장성을 고려하여 라즈베리파이를 선택했습니다.

- RaspberryPi 4B(구매처 링크)
- 4GB RAM, 16GB SD 카드
- raspberry pi os 32bit 설치
- xrdp 이용 원격 접속하여 사용합니다.
- Arduino linuxarm 32bit 1.8.15 인터넷으로 설치  
(pip이나 apt-get install에서는 더 이전 버전을 다운 받을 수 있으므로 직접 인터넷으로 다운 받는 게 좋습니다.)
- 테스트 결과 아두이노 호환보드 드라이버를 설치하지 않아도 blink 예제가 잘 동작했습니다. 또한 센서 2개를 연결했을 때 센서 값 또한 잘 읽어 왔습니다.

## AWS IoT

AWS에서 제공하는 IoT 관련 서비스들로 사물을 등록해 관리할 수 있습니다. 또한 MQTT 통신을 통해 사물의 연결 상태를 모니터링하고 메시지를 송수신 할 수 있습니다. 규칙을 만들어서 사용자가 정의한 특정 조건에 부합하면 다른 AWS와 연결할 수도 있습니다. 
산업용 IoT에서는 수 많은 센서와 장비가 사용됩니다. 이 때 AWS를 사용하면 사물의 정보를 쉽게 알 수 있을 뿐 아니라 사물을 그룹화하여 firmware를 원격으로 업데이트 하는 등 관리에 용이하기 때문에 AWS IoT를 선택했습니다.
뿐만 아니라 AWS를 사용해보는 좋은 경험이 될 거라 생각 헀기 때문에 선택하기도 했습니다. 자세한 내용 및 예제, 실습의 내용은 제 블로그의 [AWS IoT 카테고리의 글](https://dongwon18.github.io/categories/#aws-iot)의 참고해주세요. 

- DynamoDB  
센서에서 읽어들인 값을 실시간으로 데이터베이스에 저장하기 위해서 사용합니다. DynamoDB에 저장된 값은 아마존 storage인 S3로 손쉽게 보내기가 가능합니다. S3에 있는 데이터는 머신러닝, 딥러닝 등에 사용하도록 연결할 수 있기 때문에 추후 확장성을 생각하여 해당 DB를 선택했습니다.
- Lambda  
EC2, 서버용 컴퓨터 등을 따로 구비하지 않아도 필요할 때 함수를 실행해서 연산을 진행하도록 할 수 있습니다. 따라서 비용을 많이 아낄 수 있죠. 이 프로젝트에서는 관리자에게 보내는 메일의 내용을 다듬을 때 사용할 예정입니다.
- Simple Notification Service  
어플리케이션 푸시 알림, SMS 메시지, 메일 등을 보내주는 AWS 입니다. 이 프로젝트에서는 관리자에게 메일을 보낼 때 사용할 예정입니다.

## Android Application

이전까지 어플리케이션은 App inventor로 간단하게만 다뤄봤기 때문에 거의 모든 내용을 다른 분의 블로그를 참고할 예정입니다.  **AWS IoT에서 값을 가져가 안드로이드 앱으로 연결하는** 3가지 방법을 찾아봤습니다. 
사실 라즈베리파이에서 값을 `구글 파이어베이스`를 이용해서 app inventor에서 실시간으로 값을 확인하는 방법은 이전에 해봤지만 AWS를 사용해보자는 측면에서 이 방식은 배제했습니다.
현재는 Android Studio를 사용하여 앱을 만드는 것을 목표로 하고 있습니다.

- Android Studio  
AWS IoT Core에서 처리한 후 DynamoDB에 저장되어 있는 값을 앱에서 읽는 방식으로 실제 센서 측정 시간과 시간 차가 있을 수 있습니다.  
    - Mobile Hub  
    DynamoDB의 값을 읽어서 Android Studio로 보내줍니다.  
- Node-RED  
MQTT 통신 자체를 읽어오는 방식으로 Node-RED 앱을 안드로이드에 깔고 해당 MQTT 주제를 구독하여 raw 데이터를 확인하는 방식입니다. 완성된 어플리케이션으로 보기는 어렵긴 하나 test bed 프로젝트이므로 만약 시간이 넉넉치 않다면 이 방식을 사용할 예정입니다.  
- Flutter  
Android Studio와 동일하게 DynamoDB에 저장되어 있는 값을 앱에서 읽는 방식입니다. 이 때는 `Lambda 함수`를 이용하여 값을 가져옵니다.  

---

# 수정 사항

제가 중간에 아두이노를 터트려 버렸으나 다시 사기에는 남은 기간이 애매해서 라즈베리파이에서 직접 센서의 값을 읽어오기로 했습니다. 사건의 발단은 이렇습니다.

- 라즈베리파이와 아두이노를 연결해서 아두이노에서 센서의 값을 불러오던 중
    - 빵판 사용하기가 귀찮아서 아두이노 핀 중 5V 2개, GND 2개와 아날로그 핀, 디지털 핀에 각각 센서를 연결
    - MQ 3 센서 값은 읽어지나 DHT22는 읽어지지 않음
    (센서에 전원이 안 들어오던데 아마도 5V 핀 2개를 사용하면서 전류 부족 등의 문제 있었을 것 같다.)
    - 전원 부족이라 판단하고 라즈베리파이와 USB로 연결되어 있는 상태에서 12V 외부 전원을 공급함

    → 아두이노 보드가 터짐

- 현상
    - 아두이노에 코드를 업로드하려고 할 때 에러 발생
        - verification error: content mismatch
        찾아보니 부트로더가 파괴되었을 때 발생하는 현상이라고 합니다.
- 원인
    - USB와 연결되어 있을 때 12V 외부 전원을 공급하면 안된다고 하더라고요. 저처럼 태워 먹는 분이 없길 바랍니다ㅠㅠ
- 해결 방안
    - 부트로더를 복구하면 됩니다. 부트로더 복구 중 가장 기본적인 방법은 아두이노 2개를 연결하는 방식입니다. 저는 아두이노가 하나 밖에 없기 떄문에 찾아보니 컴퓨터를 이용하여 부트로더를 새로 입히는 방법도 존재하더라고요.
        - [컴퓨터를 이용하여 아두이노 부트로더 입히기](https://robodream.tistory.com/59)

        블로그에서 알려주신대로 프로그램에서는 성공했다고 나오는데 다시 아두이노 코드를 업로드 해보려 하니 똑같은 에러가 발생했습니다. 아무래도 하드웨어 자체가 파괴된 것 같아서 미련 없이 아두이노를 보내드렸습니다.

결론적으로 라즈베리파이는 아날로그 값을 읽을 수 없으므로 DHT22 센서 1개를 라즈베리파이에 연결하여 값을 읽는 방식으로 변경했습니다.

변경된 전체 아키텍처 그림은 다음과 같습니다.
  
<p align = "center">
  <img src = "/assets/images/Copy of AWS IoT Architecture modified.png" alt = "modified total architecture"> <br/>
  아두이노 없이 구성한 전체 아키텍처
</p>


