---
title: "AWS Amplify 이용 DynamoDB 데이터 웹페이지에서 검색"
excerpt: "AWS Amplify, API Gateway, AWS Lambda, DynamoDB 이용 및 안드로이드 스튜디오 관련 실패 경험들"
date: 2021-08-23
categories: 
  - IoT
tags:
  - Tutorial
  - AWS_Amplify
---
안녕하세요. 센서에서 읽어온 값을 굳이 AWS DynamoDB Console에 접속하지 않고 편리하게 사용하는 방법이 있다면 모니터링에 많은 도움이 될 것 같습니다. 센서 값 뿐 아니라 추가적인 사항을 함께 나타내기에도 좋을 것 같고요. 다양한 방법을 시도해보고 여러 번 실패 후 드디어 성공한 부분이 있어 정리해보겠습니다.

# 실패했던 시도

처음에는 안드로이드 앱에서 값을 확인할 수 있도록 하면 좋겠다고 생각해서 안드로이드 스튜디오로 앱을 만드는 방식 위주로 찾아봤습니다.

- AWS Mobile Hub, 안드로이드 스튜디오를 이용하여 DynamoDB 값 불러오기
    - 다른 분께서 하신 예제가 있어서 쉽게 따라할 수 있을 거라 생각했는데 이럴 수가... Mobile Hub가 Amplify로 대체되었다고 하더라고요. 그래서 시작부터 막힌 시도였습니다.
- AWS Amplify, 안드로이드 스튜디오를 이용하여 java로 앱 만들기
    - 여러 가지 예제를 찾아보다 GraphQL API를 이용해서 애완동물 정보를 DynamoDB에 저장하는 예제를 따라해보다 생각보다 친절하지 않아서 혼자서 AWS Amplify를 헤매 보았습니다. AWS IoT와 달리 자습서가 친절하지 않은 대신 서비스 자체에서 설명을 친절하게 하더라고요. 따라서 그대로 진행했는데...isReadOnly 가 undefined 되었다는 에러를 주면서 실행이 불가능했습니다. (지금 생각해보면 alt+enter 를 이용해서 관련 라이브러리를 import 할 수 있었을 거 같기도 한데 안드로이드 스튜디오 처음 쓰는 상황에서 아무것도 몰랐기 때문에 다른 방식을 택했습니다.
- AWS Amplify, 안드로이드 스튜디오를 이용하여 Kotlin으로 앱 만들기
    - 예제가 친절했지만... Kotlin 플러그인이 안드로이드 스튜디오 4.1부터 변경되었는데 해당 예제는 이 부분이 적용되지 않은 예제였습니다. 그래서 이곳 저곳에서 계속 에러가 나더라고요. 해당 사항을 고쳤고 에뮬레이터 실행까지 성공했지만 로그인 화면이 안 뜨고 에러가 계속 발생해서 결국 다른 방식을 선택했습니다.

# 그 외 가능한 방법

- PyQt를 이용하여 GUI를 만들고 python으로 DynamoDB의 값을 받아와서 GUI에 띄우기

    → 비교적 간단하고 이전에 firebase에서 값을 가져와 본 적이 있지만 PC에서만 확인할 수 있기보단 앱이나 웹을 이용하는 게 편리할 거라 생각했고 이전에 해 본 걸 또 해본다면 학습 효과가 떨어질 거라 생각해서 진행하지 않았습니다.

- Node-RED 앱을 이용하여 MQTT 주제를 구독해 값 실시간으로 확인하기

    → Delay가 없다는 측면에서 더 효과적일 것 같기는 하나 Node-RED 앱을 통해 값을 확인하는 것이기 때문에 앱이나 웹을 직접 다루는 것보다 간단할 것이라 생각하여(이번 기회에 전체적인 웹이나 엡 흐름이라도 이해하자는 목표를 가지고 있었습니다.) 앱이나 웹 만들기를 진행해보고 완성도가 지나치게 떨어지거나 구현 실패할 경우 차선책으로 생각한 방안입니다.

# 목표

- DynamoDB에 저장되어 있는 값을 검색하여 웹페이지에서 센서 값을 확인할 수 있다.
- AWS Amplify를 이용하여 구현한다.

# 사전 정보

- [센서 데이터 실시간으로 DynamoDB에 저장하기](https://dongwon18.github.io/sensormonitoring/save-sensor-data-to-DynamoDB/#dynamodb-%ED%85%8C%EC%9D%B4%EB%B8%94)   
이 실습에서 사용한 DynamoDB는 이전 실습에서 센서 데이터를 저장했던 DynamoDB입니다. 따라서 테이블의 상세한 내용은 이전 글을 참고해주세요.
- 저는 이전까지 웹을 거의 다뤄본 적이 없습니다. (이 블로그에 필요한 기능을 다른 분의 예시를 보고 참고할 정도입니다.) 따라서 이 실습에서 사용된 html, api 배포, gateway 등은 대부분 AWS Amplify 자습서의 내용을 참고하였습니다.

# 사용한 AWS

- AWS Amplify
- API Gateway
- AWS Lambda
- DynamoDB

# 전체적인 흐름

1. AWS Amplify를 이용하여 만든 웹페이지에서 센서 값을 알기 원하는 `timestamp` 와 `client_Id` 를 입력받습니다.
2. DynamoDB에서 원하는 값을 읽어올 수 있는 코드를 python으로 작성하여 Lambda 함수를 만듭니다.
3. API Gateway를 이용하여 `POST` 요청이 왔다면 Lambda 함수를 실행하여 DynamoDB에서 해당 값을 읽어와 응답해줍니다.
4. AWS Amplify를 이용하여 만든 웹페이지에 응답 받은 결과를 띄웁니다.

# 튜토리얼

리전은 모두 서울로 통일해 진행하였습니다.

## AWS Amplify 설정

1. `AWS Amplify 콘솔` 에서 `New app -> Host web app` 에서 깃허브를 선택해줍니다.   
저는 AWS IoT 테스트베드 레포를 선택했습니다. 해당 프로젝트와 연결할 예정이거든요. 자습서에서는 `Deploy without Git Provider` 를 선택하라고 합니다. 제가 진행해본 바로는 index.html을 어디에 저장하나 정도의 차이라서 편한대로 하시면 될 거 같습니다.
2. Github에서 선택한 레포지토리에 html  파일을 저장합니다.  
  (`YOUR-API-URL_Address` 부분에는 이후에 API Gateway에서 얻은 URL을 넣어주세요.)

<script src="https://gist.github.com/dongwon18/94ad998b22c3dbeff715377d773cdbcf.js"></script>

- 이 코드는 대부분 AWS 자습서를 참고했습니다.
- 웹페이지에 나타나는 text를 조금 변경했습니다.
- API로 넘겨주는 값 부분인 `callAPI` 함수를 수정했습니다.    
  특히 timestamp는 숫자로 DynamoDB에 지정되어 있어서 Number로 받아왔음에도 string으로 변경되어 처리되더라고요. 이것 때문에 `undefined` 라는 에러를 만났었는데 Number로 cast 해주니 해결되었습니다. 매우 허무하더라고요. 

## Lambda 함수 생성

1. `AWS Lambda 콘솔` 에서 새로운 함수를 생성합니다.
    - `새로 작성` 선택
    - 함수 이름: HelloWorldFunction(마음대로 하셔도 됩니다.)
    - runtime: python 3.8
2. 함수의 코드를 다음과 같이 입력합니다.


<script src="https://gist.github.com/dongwon18/016a7d559e0def41449853510d33d056.js"></script>


3. 테스트를 이용하여 함수가 제대로 만들어졌는지 확인합니다.

```json
{
	"timestamp": 1628829592848,
	"client_Id": "RPI"
}
```

를 `Configure test event` 에 작성하여 DynamoDB에서 제대로 값을 가져오는지 확인합니다.(timestamp는 제가 테이블에 존재하는 거 임의로 가져왔습니다.)

## API Gateway

1. `API Gateway 콘솔` 에서 새로운 API를 생성합니다.
2. `REST API` 구축을 선택합니다.
    - API 이름: HelloWorldAPI(마음대로 하셔도 됩니다.)
    - 엔드포인트 유형: 엣지 최적화

        분산된 클라이언트, 즉 퍼블릭 서비스에 접근하는 인터넷 상황에서 사용하기 좋은 옵션이라고 자습서에 명시되어 있어 선택했습니다.

    - API를 생성합니다.
3. `리소스 -> 메서드 생성 -> POST` 를 선택하고 체크를 클릭합니다.
    - 통합 유형: Lambda 함수
    - 함수: HelloWorldFunction

    꼭 함수를 먼저 만든 후 이 과정을 진행해야 권한에서 꼬이지 않습니다. 저는 이 과정을 먼저 진행하고 같은 이름으로 함수를 만들었음에도 적용이 안되더라고요. 만약 저처럼 순서를 섞어 하신 분이 있다면 해당 메서드를 삭제 후 다시 만들어주세요.

4. POST 메서드를 선택한 채로 `CORS 활성화` 에서 `POST` 만 선택되도록 설정을 변경 후 파란 버튼을 눌러 수정을 완료합니다.
5. `API 배포` 를 선택하고 새 스테이지에 이름을 아무렇게나 지어주고 배포합니다.
6. `스테이지` 에 나타나는 URL은 차후 API 호출에 사용되므로 복붙해두세요.   
참고로 이 URL을 클릭하면 `{"message":"Missing Authentication Token"}` 라고 뜹니다. 찾아보니 경로를 스테이지 변수 뒤에 더 추가해줘야 한다는데 저는 POST의 메서드 경로가 루트 바로 밑임에도 이런 현상이 일어납니다. 필요한 권한은 이미 설정해줬고요. 저는 이 문제를 해결하지 않고도 목표대로 구현되었습니다.
7. `리소스 -> POST` 에서 번개 모양을 클릭하여 해당 API를 테스트해볼 수 있습니다. 테스트는 Lambda 함수를 테스트할 때 사용했던 json 타입 입력값을 그대로 사용합니다.

## DynamoDB에 IAM 정책 생성

저는 만들어 놓은 테이블을 사용했으므로 정책만 연결해주면 됩니다.

1. `AWS DynamoDB 테이블` 에서 해당 테이블을 선택합니다.
2. `구성 -> 권한 -> 실행 역할` 의 `편집` 선택
3. `json` 선택 후 아래와 같이 변경 후 저장

<script src="https://gist.github.com/dongwon18/0e599a76737dd11b8be5ba497a413d6f.js"></script>


   - `Resource` 부분은 해당 DynamoDB의 ARN 복붙하기

## AWS Amplify로 배포

- 여기까지 하면 AWS Amplify에서 `프로비저닝 → 빌드 → 배포 → 확인`을 완료할 때까지 기다립니다.

## 결과

<p align = "center">
  <img src = "/assets/images/WEB.PNG" alt = "result from Webpage"> <br/>
  완성된 웹페이지 맟 결과
</p>

- 기다린 후 생성되어 있는 URL로 접속하면 timestamp와 client_Id를 입력하는 칸이 있는 웹페이지를 확인할 수 있습니다.

<p align = "center">
  <img src = "/assets/images/WEB2.jpg" alt = "accessing the web with phone"> <br/>
  스마트폰에서 웹사이트 방문
</p>

- AWS Amplify 덕분에 스마트폰 데이터를 연결한 상태로 해당 주소에 접속해도 같은 페이지를 볼 수 있습니다. 즉 로컬 페이지가 아니고 언제든지 확인할 수 있다는 거죠.

# 결론

- 헤맨 끝에 AWS Amplify, API Gateway, AWS Lambda를 사용하여 DynamoDB의 값을 웹 페이지에서 검색할 수 있도록 했습니다.

## 보완할 점

- 현재 누구나 페이지에 접속하여 값을 검색하고 확인할 수 있습니다. 이렇게 되면 보안에 많은 문제가 있을 수 있기 때문에 로그인 등의 방법을 통해 특정 사용자만 접속할 수 있도록 설정해야 합니다.
- 이 실습에서는 timestamp를 지정해야만 값을 확인할 수 있습니다. 이 같은 방법을 사용하면 항상 timestamp를 알아내야 하는 단점이 있습니다. 해결 방안과 단점을 정리해보자면 이렇습니다.
    - date를 입력 받고 Lambda 함수에서 해당 date의 timestamp로 변환하기
        - 정확한 시간을 초 단위로 알아야 값을 확인 가능
    - 실행 시간을 기본값으로 설정하고 DynamoDB에서 값 불러오기
        - 실행 시간과 센서에서부터 DynamoDB에 저장되는 시간까지의 차이로 인해 error 발생 가능
    - 해당 Client_Id의 값을 리스트 등을 이용하여 한 번에 확인할 수 있도록 하기
        - 많은 양의 데이터를 매번 주고 받아야 해서 비효율적
        
<div class = "notice--primary" markdown="1">
**해결책**
제가 생각하는 가장 효과적인 방법은 client id만 입력 받고 해당 id의 DynamoDB에 저장되어 있는 제일 최신 데이터만 새로 고침을 누를 때 불러오는 것입니다. 어떻게 구현할지는 더 찾아봐야 할 것 같습니다.
</div>


# 참조

- [파이썬으로 DynamoDB 값 읽어오기](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/GettingStarted.Python.03.html#GettingStarted.Python.03.02)(AWS DynamoDB Developer Guide)
- [AWS Amplify로 유니콘 웹 앱 만들기](https://aws.amazon.com/ko/getting-started/hands-on/build-web-app-s3-lambda-api-gateway-dynamodb/module-one/?e=gs2020&p=build-a-web-app-intro)(AWS 자습서)
- [AWS Amplify, 안드로이드 스튜디오, Kotlin 이용 앱 만들기](https://aws.amazon.com/ko/getting-started/hands-on/build-android-app-amplify/module-one/)(AWS 자습서)
- [AWS Amplify, DynamoDB, Lambda, API Gateway 이용 웹 만들기](https://aws.amazon.com/ko/getting-started/hands-on/build-web-app-s3-lambda-api-gateway-dynamodb/module-one/?e=gs2020&p=build-a-web-app-intro)(이 실습에서 참고한 자습서)
