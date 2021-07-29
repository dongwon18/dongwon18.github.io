---
title: AWS IoT 시작하기(IAM 생성, 사물 등록, SDK 설치
categories:
  - Tutorial
  - AWS_IoT
---

안녕하세요. 이번에는 AWS IoT 서비스를 이용하여 IoT 디바이스를 제어하는 프로젝트 중 튜토리얼 부분을 포스팅할 예정입니다. 물론 AWS 자습서에 아주 상세하게 나와있지만 따라 하면서도 설명이 조금 더 필요했던 부분이나 에러가 있었던 부분이 있어서 그 부분도 함께 설명해 볼 예정입니다.

# AWS IoT란

AWS IoT에서는 다양한 IoT 디바이스를 등록, 관리할 수 있습니다. 디바이스와의 통신을 통해 AWS의 다른 서비스와 연결할 수도 있습니다. 

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127452730-ce3ee834-2f22-44cb-bb25-96c003ea272a.png" alt = "AWS IoT 설명"> <br/>
  출처: AWS IoT 튜토리얼
</p>

- 제어 장치: R G B 의 버튼을 선택해서 밑 전구에 해당 색의 불이 들어오도록 설정하는 제어 장치입니다.
- AWS IoT: 구름 모양으로 표시된 부분입니다. AWS IoT는 다음과 같은 역할을 합니다.
    - 제어장치와 전구를 연결해서 명령을 분석 및 전달해줍니다.
    - rule을 이용하여 파란 버튼을 눌렀을 때 초록색이 켜지도록 설정을 전구에 명령을 내립니다.
    - 빨간 버튼을 누르면 빨간 전구가 켜지고 SNS, AWS Lambda, DB에 값을 저장합니다.
    - 디바이스 섀도우를 제공하여 전구의 전선이 끊어지는 등의 현상으로 인해 빨간색이 켜지는 명령을 전달하지 못했다면 디바이스 섀도우에 전구의 이상적인 상태를 저장하고 있다가 물리적인 전구의 전선이 다시 연결되는 대로 디바이스 섀도우에 저장된 모든 명령을 물리적 전구에 전송합니다.
    - 모바일 앱은 REST API 통해 디바이스 섀도우와 연결되어 전구의 상태를 확인하고 조절합니다. (섀도우와 연결되기 때문에 만약 전구의 전선이 끊어진 상태에서 앱에서 전구의 불을 초록색으로 변경한다면 앱에서 확인하는 전구의 상태는 초록색입니다. 그러나 물리적인 전구는 꺼진 상태입니다.)

위 예제에서는 사물 및 그룹에 대해서는 언급하지 않았지만 AWS IoT에서는 사물을 등록하고 사물에 대해 그룹을 생성하여 그룹에 대해 작업을 수행할 수도 있습니다. 사용되는 센서의 수가 매우 많은 산업용 IoT에 적합하다고 얘기하는 것도 이 때문인 듯 합니다. 

# AWS IoT 기본 튜토리얼

해당 부분은 한글 기준으로 설명하겠습니다.

## AWS 계정 생성 및 IAM 설정

IAM은 AWS에서 제공하는 기능으로 추가적인 비용 없이 사용 가능합니다. IAM은 권한을 관리하는 기능을 합니다. 리눅스에서 사용자, 그룹을 만들고 권한을 설정하는 것처럼 IAM도 동일한 기능을 한다고 보면 됩니다.

1. [IAM console](https://console.aws.amazon.com/iam/home#/home)에 접속하여 `사용자` 로 들어가 `사용자 추가`혹은 `사용자 생성`을 클릭합니다. 
    - 사용자 이름: Administrator
    - 엑세스 유형: `AWS Management Console 엑세스` 선택
    - 콘솔 비밀번호: `사용자 지정 비밀 번호` 선택
    - 다음 선택
2.  권한 설정: `그룹에 사용자 추가` 선택
    - 그룹 리스트에 처음에는 아무것도 없습니다. `그룹 생성`을 클릭합니다.
    - 그룹 이름: Administrators
    - 정책 필터에서 `AWS 관리 - 직무`를 택합니다.
    - 필터되어 나온 결과 중 `AdministratorAccess`를 선택합니다.
    - `그룹 생성` 선택
    - 결과적으로 정책에 대한 요약 내용이 길게 나옵니다. 그리고 다음으로 넘어가는 버튼이 없습니다. 사용자를 만들다가 그룹을 만들면서 사용자 만들기가 중지되고 그룹만 만들어진 상태입니다. 따라서 다시 사용자를 만들어야 합니다.
3.   `사용자`로 이동합니다.
    - 1번처럼 사용자 생성을 진행하다 그룹 선택에서 2번에서 만든 그룹을 선택하고 다음을 클릭합니다.
    - 사용자가 최종적으로 생성될 때까지 다음을 클릭하면 됩니다.

저는 자습서를 따라하면서 위의 순서대로 했는데 조금 더 편하게 하기 위해서는 그룹을 먼저 생성하고 사용자를 생성하는 게 좋을 것 같습니다. 

## AWS IoT에 디바이스 연결

이 부분은 `알아보기 → AWS IoT에 연결 → 디바이스 온보딩`부분을 이용하면 손쉽게 가능합니다. 저 같은 경우에는 이 방식을 통해서도 진행해봤지만 아주 친절하고 한글로 잘 되어 있어서 시키는대로 진행하면 됩니다. 저는 튜토리얼이 아닌 직접 연결하는 방식으로 진행해보겠습니다. 

- 사용 디바이스: RaspberryPi 4B
- OS: raspberry pi os 32bit

### 환경 세팅

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install cmake
sudo apt-get install libssl-dev
```

을 모두 실행해 필요한 라이브러리를 설치합니다.

```
git --version
python3 --version
pip3 --version
```

을 이용하여 git, 파이썬(3.5 버전 이상)과 pip이 설치되어 있는지 확인합니다. 라즈베리파이 os 같은 경우 이미 설치되어 있더라고요.  만약 설치되어 있지 않다면 `sudo apt-get install git` , `sudo apt-get install python3` , `sudo apt install python3-pip` 을  통해 설치해줍니다. 

### AWS IoT Device SDK for Python 설치

home 디렉토리로 이동하여 아래 명령어로 설치를 진행합니다. 

```
cd ~
python3 -m pip install awsiotsdk
git clone https://github.com/aws/aws-iot-device-sdk-python-v2.git
```

### 인증서 설치

이 때 region을 서울로 선택하지 않으니까 MQTT 연결하는 예제에서 연결할 수 없다는 오류가 발생하더라고요. 그러니까 지역을 서울로 선택했는지 꼭 확인하세요. 저는 노트북에서 해당 과정을 진행 후 인증서 파일을 라즈베리파이에 메일로 보내서 사용했는데 라즈베리파이에서 직접 이 과정을 진행해도 상관 없습니다. 

1. `보안 -> 정책 -> 생성` 을 클릭해 새로운 정책을 만듭니다.
    - 이름: RPI_Policy(마음대로 지으셔도 됩니다)
    - 작업: iot:Connect,iot:Receive,iot:Publish,iot:Subscribe
    - 리소스 ARN: *
    - 효과: `허용` 선택
    - 생성 선택
2.  `관리 → 사물 -> 사물 생성` 선택
    - `단일 사물 생성` 선택
    - 사물 이름: RPI(원하는 대로 지어도 됩니다.)
    - 다음 선택
    - `새 인증서 자동 생성(권장)` 선택 → 다음 선택
    - 아까 만든 정책 선택 → 사물 생성 선택
    - 인증서 다운로드 페이지가 나오는데 이 페이지를 닫으면 인증서를 다시 다운로드 받을 수 없으므로 모든 파일(CA3 파일 제외)을 다운로드 받아야 페이지가 닫힙니다.
    - 다운로드 한 후 파일의 이름을 변경해야 합니다.
        - xxxx-certificate.pem.crt → device.pem.crt
        - xxxx-public.pem.key → 이 예제에서는 사용 안 하므로 그냥 둡니다.
        - xxxx-private.pem.key → private.pem.key
        - AmazonRootCA1.pem → Amazon-root-CA-1.pem

    ```python
    cd ~
    mkdir certs
    ```

    을 통해 새로운 디렉토리를 만든 후 certs 디렉토리에 이름을 변경한 인증서 파일을 저장합니다. 저는 노트북에서 저장한 후 메일로 이 파일들을 보내서 라즈베리파이에서 다운 받아 해당 폴더에 저장했습니다. 처음부터 라즈베리파이에서 진행해도 상관 없습니다.

3. `설정` 선택
    - 디바이스 데이터 엔드포인트에 적혀있는 엔드포인트를 확인해주세요.

### 예제 실행

```
cd ~/aws-iot-device-sdk-python-v2/samples
python3 pubsub.py --topic topic_1 --root-ca ~/certs/Amazon-root-CA-1.pem --cert ~/certs/device.pem.crt --key ~/certs/private.pem.key --endpoint your-iot-endpoint
```

에서 마지막 `your-iot-endpoint` 에 아까 확인한 엔드포인트를 대신 넣어주세요. 그리고 엔터 치면 예제가 실행됩니다!

### 결과

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127453019-a78e0dfe-fd16-43ad-92f3-7a52337ba06b.png" alt = "예제 결과"> <br/>
  예제 결과
</p>


MQTT를 통해 메시지를 송수신하는 결과입니다. `ctrl + c` 를 이용하면 프로그램을 중지할 수 있습니다. 이를 통해 SDK가 잘 설치되었다는 걸 예제를 통해 확인할 수 있었습니다.  

다음에는 AWS IoT rule를 생성하여 MQTT를 이용해 json 파일 형식으로 데이터를 주고 받은 예제를 설명해보겠습니다.
  
# 참고
- [AWS IoT tutorial](https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html) `What is AWS IoT?` ~ `Connecting to AWS IoT Core`

글이 도움이 되었다면 댓글로 알려주세요^^
