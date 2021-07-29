---
title: htm.core Google Colab에 설치(Install htm.core to Google Colab)
date: 2021-07-29
categories:
  - MachineLearning
  - Tutorial
toc: true
toc_sticky: true
---

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127414313-aa55cd92-0002-46c2-a834-b603a8a83ffd.png"> <br/>
  이미지 출처: blog.tensorflow.org
</p>

안녕하세요. 저번에 얘기 드린 대로 htm.core를 Google Colaboratory에 설치할 수도 있습니다. 아직 뚜렷한 활용 방안을 찾지는 못했지만 일단 잊어버리기 전에 정리하는 차원에서 글을 작성해봤습니다.

## 환경

-   Google Colab
    -   Google Colab은 파이썬 3.7 이상 버전을 지원하고 있기 때문에 Nupic 대신 htm.core를 설치합니다.
    -   Google Colab에는 환경이 거의 갖춰져 있기 때문에 특별히 신경 쓸 사항은 없습니다.
    -   설치를 하고 연결이 끊기기 전에 빠르게 돌려보고 싶었던 데이터셋을 돌려봐야 합니다. 연결이 끊기면 설치한 게 다시 초기화되어 없어지니까요.
    -   Google Colab은 리눅스 환경에서 구동됩니다.

## 설치 방법

코랩에서 cmd처럼 명령을 내릴 수 있도록 매직 명령어를 사용합니다. (앞에 느낌표 '!'를 붙이면 cmd에서처럼 명령어를 사용할 수 있습니다.)

1.  ```
    !git clone https://github.com/htm-community/htm.core
    ```
    
    를 이용하여 레포지토리를 복사합니다.
2.  ```
     !python htm.core/setup.py install
    ```
    
    로 설치를 진행합니다.
3.  ```
     !python htm.core/setup.py test
    ```
    
    를 이용하여 제대로 설치되었는지 확인합니다. 저는 결과적으로 아래와 같이 4개의 검사는 진행되지 않았습니다.  
    
    
<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127414432-91aa35ea-1eca-4517-8456-8fa8c4ec05d6.png" alt = "test result"> <br/>
  테스트 결과
</p>

4.  바로 코드 블록에서 `import htm help(htm)`을 입력하면 htm 라이브러리가 없다는 에러가 나옵니다. 아무래도 코랩에서 노트북을 실행하는 환경과 제가 설치한 환경이 달라서 생기는 문제가 아닐까 추측해봅니다.와 같이 매직 명령어로 파이썬을 실행한 후 입력하면 잘 실행됩니다.

5.  ```
      !python 
      >>>import htm 
      >>>help(htm)
    ```

   

6.  결론적으로 5번과 같이 매직 명령어로 파이썬을 노트북 내에서 실행해야지만 htm 라이브러리를 사용할 수 있는데요. 저는 이 때문에 코랩에 htm.core를 설치한 후 활용 방안을 아직 찾지 못하고 있습니다. 만약 찾게 되신다면 꼭 알려주시길 바랍니다!

## 결론

이와 같이 코랩에 htm.core를 설치할 수 있습니다만 말씀드렸다시피 활용 방안은 아직 찾지 못했습니다. 로컬 개발 환경에 설치하고 싶으신 분들은 제 [이전 글](https://dongwon18.github.io/machinelearning/tutorial/htm-core-install/)을 참고하면 도움이 될 것 같습니다.

## 참고

```
!python -m pip install -i https://test.pypi.org/simple/ htm.core
```

의 경우 pytest의 버전이 맞지 않는다는 에러 메시지를 띄워서 source를 사용하는 방식을 택했습니다.
