---
title: "AWS IoT Shadow(Shadow란?, 예제)"
excerpt: "섀도우 개념, AWS IoT 예제 따라하기"
date: 2021-08-04
categories: 
  - AWS_IoT
  - Tutorial
---

안녕하세요. 지금까지 AWS IoT에서 1.  사물 등록 2. Rule, Lambda function 사용해서 특정 상황에서 메일 보내기 3. Rule 이용해서 DynamoDB에 값 저장하기 예제를 진행해왔습니다. 만약 이전 글이 궁금하신 분은 [AWS_IoT 카테고리](https://dongwon18.github.io/categories/#aws-iot)에서 확인하실 수 있습니다.   
이번에는 Shadow 기능을 이용할 수 있는 예제를 진행해볼 예정입니다.

# Shadow (섀도우)란?

[좋은 Cloud IoT 구성 방법](https://dongwon18.github.io/aws_iot/theory/Well-Architected-Iot-Framework/) 에서도 언급했었지만 디바이스는 다양한 이유로 클라우드에서 보낸 명령을 즉각적으로 이행하지 못할 수 있습니다. 인터넷 연결이 끊겼거나 디바이스 전원이 꺼졌다면 당연히 명령을 이행하지 못하겠죠. 예를 들어 보겠습니다.

> 클라우드에서 빨간 LED를 점등하라고 명령을 내렸는데 디바이스는 꺼져 있는 상황을 가정해봅니다. 클라우드에서는 <span style="color:red">빨간 LED</span>가 점등된 것으로 기록이 되어 있겠지만 디바이스에서는 해당 명령을 받지 못했습니다. 따라서 여전히 이전 상태 그대로 <span style="color:green">초록 LED</span>를 점등하고 있는 상태인거죠. 실제 디바이스의 상태와 클라우드에 기록된 상태가 다른 상황이 발생하게 됩니다. 만약 그 디바이스가 신호등이었다면? 클라우드에서 이 신호등이 초록불인지 모른 채 다른 신호등을 초록불로 변경하게 되면 정말 큰일입니다!   

생각만 해도 아찔한 이런 상황을 방지하기 위해서 AWS IoT에서는 섀도우라는 개념을 도입합니다. `클라우드에 디바이스의 상태를 그림자처럼 따라하는` 개념을 만드는 거죠. 섀도우에서는 세 가지 상태를 저장합니다. `desired,  reported, delta` 입니다. 

- desired: 사용자가 내린 명령입니다. 위의 예제에서는 `빨간색 LED를 점등해라` 에 해당합니다.
- reported: 디바이스에서 클라우드에게 보고한 현재 상태입니다. 즉 실제 디바이스의 상태라고 볼 수 있습니다. 예제에서는 `초록색 LED 점등` 에 해당합니다.
- delta: desired와 reported의 상태가 다르다면 delta는 클라우드에서 내린 명령입니다. 예제에서는 desired는 빨간색, reported는 초록색인 상태이므로 클라우드에서 내린 명령인 `빨간색 LED를 점등해라` 입니다.

위 예제에서 디바이스의 전원이 켜진다면 다시 desired와 delta를 확인하고 <span style="color:red">빨간색 LED</span>를 점등하게 됩니다. 그리고 빨간색 LED를 점등했다고 클라우드에 보고하게 되죠. (reported가 red로 변합니다.) desired와 reported가 동일해졌으므로 delta는 무효해집니다.

# 사용한 AWS
- AWS IoT
  - 사물 및 인증서 생성
  - 정책 생성
  - Shadow 사용
  - 연결은 2,250,000분, 디바이스 섀도우는 225,000건, 메시지는 500,000건이 계정 생성 후 12개월 동안 사용가능하므로 이 예제를 진행할 때 요금이 부여될 걱정은 안 하셔도 될 것 같습니다.

# 전체적인 흐름
1. MQTT 특정 주제를 디바이스에서 섀도우를 하면서 publish, receive, subscribe, connect 할 수 있도록 policy를 생성해줍니다.
2. 등록된 사물(없다면 새로 등록)에 만든 policy를 연결합니다.
3. 라즈베리파이에서 shadow.py라는 AWS IoT에서 제공하는 예제를 실행하고 결과를 확인합니다.

다른 예제보다 간단하죠?

# 튜토리얼

MQTT 통신 시 지역을 `서울`로 설정해야 동작하므로 지역을 잘 선택했는지 확인해주세요.

1. thingname, region, account 번호 알아내기
    - thingname: RPI(사용할 사물 이름으로 저는 이전 예제를 진행하면서 `RPI`라고 지어서 그대로 사용)
    - region: ap-northeast-2(서울의 region 코드)
    - account: 계정 번호(아래 사진과 같이 계정 옆 `▽` 를 클릭해 확인)

<p align = "center">
  <img src = "/assets/images/SHADOW1.PNG" alt = "finding account no."> <br/>
  account 번호 찾기(내 계정 옆에 적혀 있는 숫자)
</p>


2. AWS IoT Console → 보안 → 정책 → `생성` 클릭
    - 이름: My_Device_Shadow_Policy(마음대로 지으셔도 됩니다)
    - 설명문 추가 옆 `고급 모드` 선택

<p align = "center">
  <img src = "/assets/images/SHADOW2.PNG" alt = "create policy"> <br/>
  처음에는 기본모드로 선택되어 있어 코드를 넣을 곳이 없습니다. 고급 모드로 변경하고 적혀 있던 걸 지운 후 아래 내용을 복붙해주세요.
</p>    

   - 아래 코드에서 `thingname`, `region` , `account` 를 1에서 알아낸 걸로 대체하여 복붙

   ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "iot:Publish"
          ],
          "Resource": [
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/get",
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/update"
          ]
        },
        {
          "Effect": "Allow",
          "Action": [
            "iot:Receive"
          ],
          "Resource": [
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/get/accepted",
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/get/rejected",
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/update/accepted",
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/update/rejected",
            "arn:aws:iot:region:account:topic/$aws/things/thingname/shadow/update/delta"
          ]
        },
        {
          "Effect": "Allow",
          "Action": [
            "iot:Subscribe"
          ],
          "Resource": [
            "arn:aws:iot:region:account:topicfilter/$aws/things/thingname/shadow/get/accepted",
            "arn:aws:iot:region:account:topicfilter/$aws/things/thingname/shadow/get/rejected",
            "arn:aws:iot:region:account:topicfilter/$aws/things/thingname/shadow/update/accepted",
            "arn:aws:iot:region:account:topicfilter/$aws/things/thingname/shadow/update/rejected",
            "arn:aws:iot:region:account:topicfilter/$aws/things/thingname/shadow/update/delta"
          ]
        },
        {
          "Effect": "Allow",
          "Action": "iot:Connect",
          "Resource": "arn:aws:iot:region:account:client/test-*"
        }
      ]
    }
   ```

   - `생성` 클릭
3. 사물 만들기
저는 이미 이전 예제에서 `RPI` 라는 이름으로 생성했습니다. 아직 생성하지 않으셨다면 [이전 글](https://dongwon18.github.io/tutorial/aws_iot/start-AWS-IoT/)을 참고해서 생성해주세요.
4.  관리 → 사물 → `인증서` 클릭(만들어져 있는 긴 이름의 인증서를 클릭하면 됩니다.)
    - `정책` → `작업` → `정책 연결` 클릭
    - 아까 만든 정책 선택 후 등록

<p align = "center">
  <img src = "/assets/images/SHADOW3.PNG" alt = "register new policy"> <br/>
  새로운 정책 등록
</p>


5. 라즈베리파이에서 진행
    - 저는 이전 예제에서 SDK와 인증서도 이미 설치했기 때문에 그 부분은 생략하겠습니다. 아직 진행하지 않으셨다면 [AWS IoT 시작하기](https://dongwon18.github.io/tutorial/aws_iot/start-AWS-IoT/) 글에서 해당 부분을 진행해주세요.
    - your-iot-thing-name: RPI(만든 사물 이름)
    - your-iot-endpoint: AWS IoT 콘솔 → 설정 → 엔드포인트의 값
    - cmd에 다음과 같이 입력합니다. 이 때 위에서 찾은 사물 이름과 엔드포인트 값을 대체하여 넣어줍니다.

    ```
    cd ~/aws-iot-device-sdk-python-v2/samples
    python3 shadow.py --root-ca ~/certs/Amazon-root-CA-1.pem --cert ~/certs/device.pem.crt --key ~/certs/private.pem.key --endpoint your-iot-endpoint --thing-name your-iot-thing-name
    ```
    
## 결과 확인

1. AWS IoT 콘솔 → 테스트
2. `주제 구독` 에서 `$aws/things/thingname/shadow/update/#` 구독(thingname은 사물 이름으로 변경)
3. 라즈베리파이에서 `Entered desired value` 에 red, yellow, green 중 원하는 것 입력하고 결과 확인

<p align = "center">
  <img src = "/assets/images/SHADOW4.PNG" alt = "executing shadow.py"> <br/>
  cmd 창 결과물
</p>

4. AWS IoT 콘솔 테스트에서 구독한 주제에서 `desired, reported` 확인 가능

<p align = "center">
  <img src = "/assets/images/SHADOW5.PNG" alt = "result in AWS IoT console"> <br/>
  AWS IoT 콘솔 테스트에서 확인한 결과
</p>


5. AWS IoT 콘솔 → 관리 → 사물 → 사물 이름 선택 → 디바이스 섀도우 → 클래식 섀도우를 클릭하면 Shadow state document와 Medadata 확인 가능
6. `편집` 누르고 desired의 값을 원하는 값으로 변경하고 업데이트하면 디바이스의 설정값을 클라우드에서 변경 가능

<p align = "center">
  <img src = "/assets/images/SHADOW6.PNG" alt = "modifying shadow document"> <br/>
  AWS IoT shadow statement document에서 수정하기
</p>

7. 만약 디바이스의 전원이 꺼져 있다면 다음과 같이 `delta` 에 desired의 값이 표시됨

<p align = "center">
  <img src = "/assets/images/SHADOW7.PNG" alt = "delta"> <br/>
  delta에 red 표시됨
</p>

# 결론

이번 글을 통해 섀도우가 무엇인지, 왜 필요한지를 알아봤습니다. 또한 AWS IoT에서 제공하는 예제를 통해 섀도우를 라즈베리파이에서 실행하는 방법을 알아봤습니다. 디바이스의 상태를 디바이스, 클라우드 두 곳에서 모두 변경할 수 있었으며 변경한 상태는 섀도우에 기록되어 디바이스가 즉각적으로 명령을 시행할 수 없는 상태에도 대비할 수 있었습니다.  


이것으로 기본적인 AWS IoT 예제는 끝이 났습니다. 👏👏 다음부터는 제가 진행하고 있는 라즈베리파이 이용 센서 데이터를 읽어와 AWS IoT에서 제어하는 프로젝트를 시작하려 합니다.

# 참고

- [AWS IoT 자습서](https://docs.aws.amazon.com/iot/latest/developerguide/iot-shadows-tutorial.html)

이 글이 도움이 되었다면 댓글로 알려주세요^^

오개념이 있다면 언제든지 댓글로 알려주세요><
