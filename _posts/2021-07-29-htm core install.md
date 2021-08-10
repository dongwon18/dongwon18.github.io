---
title: "htm.core 아나콘다 가상환경 설치(htm.core in Anaconda environment)"
date: 2021-07-29
categories:
  - MachineLearning
tags:
  - Tutorial
toc: true
toc_sticky: true
---

<p align="center">
  <img src="https://user-images.githubusercontent.com/74483608/127412734-299451a3-198f-465f-901a-3b3cf7cca32d.png"> <br/>
  출처: https://numenta.com/
</p>

안녕하세요. 저번에는 계층형 시간 메모리의 개념을 설명 드렸습니다. 이번에는 HTM을 파이썬이나 C++에서 사용할 수 있도록 라이브러리를 설치하는 방법을 공유하려 합니다.

If you are trying to install htm.core using your Anaconda virtual environment, following the post will be helpful. Please follow the `highlighted` part.(Other Korean is detailed explanation.) It will be enough to install htm.core library. Plus, I refered htm.core github page a lot. So, if you need any other information about the library, please read [htm.core github site](https://github.com/htm-community/htm.core).

## 환경(Environment)

저는 아나콘다 가상환경에서 설치를 진행했습니다. 설치를 할 때 항상 개발환경이 꼬일까봐 두려워하는 편인데 역시 그럴 땐 가상환경입니다. 이전에 tensorflow를 사용하기 위해 만들어놓은 딥 러닝용 가상환경에 설치를 먼저 진행해봤지만 계속 에러와 함께 설치 불가하다고 떠서 아예 새로운 개발환경을 만들어 진행했습니다.

-   Windows10 64bit
-   Anaconda3 Prompt conda 4.10.1
-   python 3.7.10
    -   (깃허브에 나와있는 내용으로는 3.7 버전 이상이면 가능하다고 하는데 issue를 보니 3.8까지는 어느 정도 호환이 되는 것으로 보입니다. 그러나 저는 딥러닝 용 가상환경 파이썬이 3.8.10인데 계속 에러가 떠서 이번에는 3.7 버전으로 설치해봤습니다.)
-   Visual Studio 2019
    -   (깃허브에는 2017도 가능하다고 했지만 저는 최신 버전인 2019를 설치했고 기본 선택으로 설치하되 'c++을 사용한 데스크톱 개발만 추가'로 선택했습니다.)
    -   만약 Visual Studio가 설치되어 있지 않다면 `WinError2 지정 파일을 찾을 수 없습니다.` 에러가 뜰 수 있습니다. (제 경우 그랬습니다.) 깃 허브 페이지에 전제 조건으로 명시되어 있지는 않지만 에러를 해결하기 위해 issue를 확인해 본 결과 2017 혹은 2019 버전을 설치해야만 합니다.
-   git
    -   설치 방식이 여러 개 있지만 시도해본 결과 source를 이용한 방식이 제일 보편적으로 설치가 잘 된다고 하여 이 방식을 사용할 예정입니다.

## 참고

-   Numenta에서 공식적으로 지원하는 nupic은 python 2.7 버전을 끝으로 유지만 할 뿐 더 이상 개발하지 않습니다. 따라서 python 3.7 이상 버전을 사용하신다면 htm.core를 설치하셔야 합니다.
    -   제 경우는 이걸 몰라서 한참 헤매다 htm.core를 알게 되어 htm.core로 다시 설치를 시도했습니다.
    -   참고로 python 3.7 이상 버전에서 nupic을 설치하려고 할 때 아래와 같은 오류가 발생합니다.(시키는 대로 unittest2 버전을 맞춰서 설치해도 해결되지 않습니다.)python 3.8.10 version에서 nupic 설치 시 오류을 통해 nupic을 설치할 수 있습니다. 자세한 사항은 [nupic 깃 허브](https://github.com/numenta/nupic)를 통해 확인해주세요.

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127412924-58471501-c515-4a15-8f9b-b95b249d197b.png" alt = "nupic install failed"> <br/>
  nupic install 실패
</p>

-   `pip install nupic`  
    을 통해 nupic을 설치할 수 있습니다.
-   htm.core는 Numenta community에서 배포한 라이브러리로 python 3.7 버전 이상을 지원하며 c++ 또한 지원합니다. 자세한 사항은 [htm core 깃 허브](https://github.com/htm-community/htm.core)를 참조해주세요.
-   `pip install htm` 을 이용하여 설치할 수 있는 라이브러리와 다른 라이브러리입니다. 저도 처음에 헤매다 이 라이브러리를 설치하게 되었는데 이 라이브러리가 설치가 되어 있다면 파이썬에서 `import htm` 을 실행할 때 htm.core 라이브러리가 아닌 다른 라이브러리가 설치되게 되니 혹시 설치하셨다면 `pip uninstall htm`으로 삭제하시면 됩니다.(What you can install with the pip command is different with htm.core, so please uninstall it to use htm.core library.)
-   Google Colaboratory에도 설치가 가능합니다. 저는 처음에 혹시 개발환경이 꼬여서 pc를 밀어야 하진 않을까 하는 걱정에 Google Colab에 먼저 설치를 해봤습니다. Google Colab에 설치는 비교적 간단했는데요. 대신 연결 시간이 정해져 있어서 만약 연결이 끊긴다면 다시 htm.core를 설치해야 한다는 치명적인 단점이 있습니다. 또 google colab에서 바로 `import htm`은 불가하고 python 실행 후 가능해서 실제로 응용하기까기 좀 더 고민이 필요한 방법이었습니다. Google colab에 설치하는 방법은 다음 포스팅에 올려 드리겠습니다. (You can install htm.core to Google Colaboratory too. It will be posted in the other posting.)

## 설치 방법

전제 조건 및 환경은 제 기준으로 설명드립니다.

1.  Anaconda Prompt를 실행합니다.(아래 모든 과정은 아나콘다 가상환경에서 실행합니다.)(All following things are done uisng Anaconda Prompt)
2.  ```
    conda activate htm
    ```
    
    만들어 놓은 개발 환경을 활성화합니다.(제 개발환경 이름은 htm 입니다.)
    
    
3.  ```
     git clone https://github.com/htm-community/htm.core
    ```
    
    을 이용하여 레포지토리를 복사합니다.
4.  ```
     cd htm.core
    ```
    
    을 이용하여 복사한 레포의 가장 root directory로 이동합니다.  
    (clone할 장소를 특별히 지정하지 않았다면 저처럼 C:\\Users\\user에 다운로드되고 최종적인 절대 경로는 C:\\Users\\user\\htm.core 입니다.)
5.  ```
     pip install -r requirments.txt
    ```
    
    으로 요구 사항을 충족시켜 줍니다.
6.  setup.py 파일을 열어 377줄의 open(os.path.join(REPO\_DIR, "README.md", "r")에 옵션으로 encoding = 'UTF-8'로 지정해줍니다. 파일을 저장해줍니다.  
   
<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127413078-d2cefdec-6a66-4c5b-9db0-7787d68dbcca.png" alt = "change setup.py">
  setup.py encoding 지정
</p>

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127413258-5e058b06-30b6-46c2-93bc-55302fa049f8.png" alt = "CP949 error">
  발생했던 encoding error
</p>

  
  깃허브 README에는 이렇게 하면 설치를 할 수 있다고 되어 있는데 저는 이대로 설치하니 UnicodeDecodeError라고 뜨더라고요. CP949 문제인데 아마도 한글을 사용하면서 CP949 encoding 스타일을 사용하다보니 이런 문제가 발생하는 것 같습니다. 해결법을 고민하다 문제가 있을 수도 있겠지만 일단 설치를 위해 setup.py 파일을 수정했습니다. 이와 같이 파일을 열 때 `encoding = 'UTF-8'`로 지정해줌으로써 문제를 해결할 수 있었습니다.

7.  ```
    python setup.py install
    ```
    
    을 통해 설치를 시작합니다. 설치는 20분 정도? 걸렸던 것 같습니다.  
    깃허브에 의하면 아나콘다 프롬프트를 사용하지 않을 경우 --user --force 같은 옵션을 추가로 지정해야 된다고 나와있습니다.
8.  아주 긴 설치 로그 후에 이와 같이 Setup complete라고 뜨면 일단 설치를 마쳤다고 볼 수 있습니다.(박수!!!!!)

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127413473-f06f183b-ca95-41f6-ab38-e99cd3cf543a.png" alt = "install success"> <br/>
  install 성공
</p>

9.  ```
    python htm.core test
    ```
    
    를 이용하여 제대로 설치가 되었는지 테스트를 진행할 수도 있습니다. 저 같은 경우 진행해보니test 결과  
    에서 볼 수 있는 것처럼 일부 문제가 있는 것 같긴 합니다.

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127413594-4f15a1d3-8122-492c-b070-ee98bca2fa82.png" alt = "test result"> <br/>
  테스트 결과
</p>
    
    
10.  설치를 했으니 파이썬에서 import 해 봐야죠? `import htm help(htm)` 을 이용하여 사용여 도움이 될 수 있는 package 정보를 얻을 수 있습니다.

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/74483608/127413707-53572e02-4b52-4711-b222-b24a488b68ff.png" alt = "help htm"> <br/>
  help(htm)
</p>



## 결론

-   저는 Visual Studio가 설치되어 있지 않아 다양한 에러를 만나면서 설치가 매우 어려웠는데 이 글이 도움이 될 수 있길 바랍니다.
-   곧 htm.core 라이브러리를 이용하여 htm 모델을 학습시키고 결과를 확인하는 글도 포스팅할 예정입니다.

## 참고 문헌

-   [htm.core Github](https://github.com/htm-community/htm.core)  
    모든 설치 방법 및 에러에 관한 사항을 참조했습니다. 만약 설치 중 문제가 발생한다면 issue에 검색하여 비슷한 문제를 겪었던 다른 사람들의 해결 방안을 볼 수 있습니다.

