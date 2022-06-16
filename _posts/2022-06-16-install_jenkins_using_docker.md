---
title: "docker로 jenkins 설치(install jenkins using docker)"
excerpt: "docker: error during connect: In the default daemon configuration on Windows, no matching manifest for windows/amd64, Error response from daemon: failed to create endpoint jenkins on network nat"
date: 2022-06-16
categories:
  - ErrorHandling
tags:
  - ErrorHandling
  - windows
  - docker
  - jenkins
---
안녕하세요. docker에 jenkins를 설치하다가 계속 에러를 만나서 다양하게 서칭해본 결과 해결했던 방법을 남겨두려고 합니다.

If you have those errors while installing jenkins using docker with windows 10,

- check if you install WSL2 Linux kernel update package
- check if your containers are running with linux containers
    - if not, change it using switch to linux containers...

Below is about what I have done to solve this problem.

## 개발 환경

- windows 10 Education version 20H2
- windows용 docker installer를 통해 기본 상황을 건드리지 않고 그대로 설치
- 모든 명령어 windows powershell 관리자 모드에서 실행

## 에러 상황

```
$ docker run -d -p 9090:8080 -p 50000:50000 -v /var/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --name jenkins -u root jenkins/jenkins:lts-jdk11
docker: error during connect: In the default daemon configuration on Windows, the docker client must be run with elevated privileges to connect.: Post "http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/containers/create?name=jenkins": open //./pipe/docker_engine: The system cannot find the file specified.
```

- 명령어를 통해 9090 포트를 사용하는 jenkins를 설치하려고 했으나  에러 만남
- 찾아본 결과 크게 다음과 같은 해결책이 존재했음
    - power shell을 관리자 권한으로 실행
        
        → 해결되지 않음
        
        참고적으로 cmd로도 해봤지만 동일함
        
    - “C:\Program Files\Docker\Docker>./DockerCli.exe” -SwitchDaemon
        - docker 파일이 설치된 폴더로 이동해 daemon 변경
        - windows를 restart하기
        
        → 새로운 에러 발생
        
- 새로운 에러(windows/amd64 관련)
    
    ```
    $ docker run -d -p 9090:8080 -p 50000:50000 --name jenkins -u root jenkins/jenkins:lts-jdk11
    no matching manifest for windows/amd64 10.0.19042 in the manifest list entries
    ```
    
    - windows/amd64에 맞는 entry가 존재하지 않는다는 에러 발생
    - docker desktop settings > docker engine 에 존재하는 json 파일의 “experimental”: true로 변경
    
    → 동일한 명령어 사용했을 때 설치는 되지만 새로운 에러 발생
    
- 새로운 에러(failed to create endpoint 관련)
    
    ```
    $ WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (windows/amd64) and no specific platform was requested
    3e76a2be7dd0d136d8c7fb4794d2a38afd89e08fa8e65734ff08aa7f0104b767
    docker: Error response from daemon: failed to create endpoint jenkins on network nat: failed during hnsCallRawResponse: hnsCall failed in Win32: The process cannot access the file because it is being used by another process. (0x20).
    ```
    
    - 대부분 이 에러는 해당 포트를 이미 다른 프로그램에서 사용하고 있기 때문에 발생한다는 의견이 많았음
        - 그러나 리소스 모니터로 보고 명령어로 사용 중인 port를 봤을 때 9090 포트를 사용하고 있는 PID가 존재하지 않음
        
        → warning의 내용에 힌트를 얻어 뭔가 빠트린 게 docker 설치부터 다시 따라함
        

## 해결 방식

- docker를 실행했을 때 warning 내용 중 `WSL install not finished`와 비슷한  문구를 봤던 게 기억남
- 결론적으로 WSL2를 설치하지 않아서 발생한 문제였음
1. 아래 명령어 power shell에서 실행

```
$ dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
$ dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

1. 윈도우 재부팅
2. [https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) 에서 WSL2 Linux 커널 업데이트 패키지 다운 받아 설치
3. 아래 명령어 power shell에서 실행
    
    ```
    $ wsl --set-default-version 2
    WSL 2와의 주요 차이점에 대한 자세한 내용은 https://aka.ms/wsl2를 참조하세요
    작업을 완료했습니다.
    ```
    
4. `작업 표시줄 > 숨겨진 아이콘 표시 > docker icon에 오른 클릭 > Switch to Linux Containers…` 를 클릭(이 때 docker desktop 화면은 닫아야 한다)
5. 실행 중인 docker 확인
    
    ```
    $ wsl -l -v
    NAME                   STATE           VERSION
    * docker-desktop         Running         2
      docker-desktop-data    Running         2
    ```
    
6. 이 상태에서 다음 명령어 실행
    
    ```
    $ docker run -d -p 9090:8080 -p 50000:50000 --name jenkins -u root jenkins/jenkins:lts-jdk11
    Unable to find image 'jenkins/jenkins:lts-jdk11' locally
    lts-jdk11: Pulling from jenkins/jenkins
    67e8aa6c8bbc: Pull complete
    d4b310eb3fdf: Pull complete
    79f2f383117e: Pull complete
    8b0534c7187a: Pull complete
    e4ebc026722c: Pull complete
    b8b2b1a015c7: Pull complete
    1308663db51f: Pull complete
    a9980750ebb2: Pull complete
    771a2a8f3665: Pull complete
    8767b747a3d4: Pull complete
    68fbb5fc468d: Pull complete
    76dd80acf4f6: Pull complete
    bb3b9e0921fc: Pull complete
    f469501b9a46: Pull complete
    c35c92bf7d6e: Pull complete
    e4260d58d0c6: Pull complete
    62d1b6acbca2: Pull complete
    Digest: sha256:9893bfe43a850a0c80a4dc6e40d212e345ce8ad5537a4cf1d3f2033787a43695
    Status: Downloaded newer image for jenkins/jenkins:lts-jdk11
    01a38077e160bf72fe286a6acdab69a96487ab21abe92671e477d93ac9aaddaa
    ```
    
    - 드디어 `[localhost:9090](http://localhost:9090)` 으로 접속하면 jenkins를 만나볼 수 있음!
7. 비밀번호는 `docker logs jenkins` 를 실행해 중간 즈음에 적혀 있는 걸로 입력하면 됨

## 결론

<div class="notice--info" markdown="1">
다른 해결 방법을 많이 고려해봤지만 결과적으로 `WSL` 설치와 `switch to linux containers...` 를 하지 않았던 게 문제였던 것 같습니다.
</div>
<p align = 'center'> 이 글이 도움이 되셨다면 댓글로 알려주세요^^ </p>

## 참고한 것

- [WSL2 설치]([https://www.lainyzine.com/ko/article/a-complete-guide-to-how-to-install-docker-desktop-on-windows-10/](https://www.lainyzine.com/ko/article/a-complete-guide-to-how-to-install-docker-desktop-on-windows-10/))
