---
title: "Docker 기초"
excerpt: "도커 특징, 도커 용어, 명령어 정리, docker-compose, github&docker 연동 "
date: 2022-09-28
categories:
  - DevEn
tags: 
  - DevEn
  - Tutorial
  - Docker
  - CI/CD
---
안녕하세요.  
'How to Get started with Docker'라는 예제를 보고 Docker 사용법을 익히며 정리했던 내용을 공유하려 합니다.

## 왜 Docker를 사용해야 하는가

- 서버를 pet 처럼 생각해야 함(실제 강연자 워딩)
    - 오래 살아야 하고 돌봐줘야 하고 아프지 않아야 함
- dependency, server 이슈 등 서버 운영 매우 힘듦

→ Docker to the Rescue

- Epheneral: 서버가 죽으면 그냥 바꾸면 됨
- build → ship → run image
- CI/CD(test & deploy without stop), different version(다른 버전은 install 없이 쉽게 사용 가능), roll forward(when defect, just start new server)

## Docker 기초

- container는 host os와 독립적으로 실행되는 process
- container image는 컨테이너에서 사용되는 isolated custom filesystem→ dependency, config, script, binary, env var, default cmd, metadata 등 모든 app 실행에 필요한 걸 포함
- docker desktop 설치할 것
- dockerfile: docker engine에서 실행을 원하는 cmd 모음
    
    ```docker
    FROM node:12.16.3
    
    WORKDIR /code
    
    ENV PORT 80
    
    COPY package.json /code/package.json
    
    RUN npm install
    
    COPY . /code
    
    CMD ["node", "src/server.js"]
    ```
    
    - FROM으로 시작
        - base image를 지정함
        - 예) node로 설정하면 node, npm 다 설정되어 있음
    - WORKDIR: working directory
    - ENV: variable(environment var 등)
    - COPY package.json을 /code/package.json에 복사
    - RUN: run 뒤에는 docker execute함
    - CMD: 시작했을 때 실행해야 할 cmd
- hub에 공유하기
    - group에게 access 할 수 있도록 하기 등 가능
    - auto build, ci/cd 가능
    - repo에 image 넣기
    - github push될 때마다 auto build 가능
    - cmd에서 docker repo에 push하면 됨
    - docker tag hello-world pmckee/hello-world로 이름 다시 지어서 올리면 됨
    - docker pull해서 img 가져올 수 도 있음

### 명령어

- docker build: 이미지 빌드
    - tag: 이름 짓기
    - file: docker 파일이 어디에 있는지 얘기
- docker run [image 이름]
    - -d: background run 하게 해줌
- docker start [container 이름]: container 실행
- docker stop [container 이름]: 해당 container 중지
    - stop을 하면 container는 유지하되 SIGTERM 보내서 run을 멈춤, 새로운 이미지에 해당 container의 상태를 저장 가능
- docker rm [이름]: 해당 container 삭제
    - container 지워버림, state 없어지므로 새로운 이미지에 저장 불가
    - 하면 docker ps -a해도 안 나옴
    - stop을 한 후에 rm 해야 함(안 그러면 zombie process 될 가능성 있음)
- docker images: 이미지 list 보기
- docker ps: 실행 중인 container list 보기
- docker ps -a: 존재하는 container list 보기
- docker rmi [image 이름]
    - img 삭제
- docker logs [이름]
    - background에서 실행되고 있는 거 log 볼 수 있음
    - -f 하면 계속 볼 수 있음

## Docker-compose

- 여러 컨테이너 관리하기 힘들 수 있음 → docker-compose.yml 사용
    
    ```docker
    # docker-compose.yml
    version: '2'
    
    services:
    	web:
    		build:
    			context:
    			dockerfile: Dockerfile
    		container_name: web
    		ports:
    			- "8080:80"
    	db:
    		image: mongo:3.6.1
    		container_name: db
    		volumes:
    			- mongodb:/data/db
    			- mongodb_config:/data/configdb
    		ports:
    			- 27017:27-17
    		command: mongod
    
    volumes:
    	mongodb:
    	mongodb_config:
    ```
    
    - service가 지금 web 하나 있음
    - context: dockerfile의 path와 비슷
    - 도커 파일 지정, container 이름과 포트 지정 가능
    - docker-compose가 있는 dir에서 docker-compose up -d 하면 background로 돌아감
    - _로 img와 container 이름 이어줌(hello-world_web)
    - docker-compose down으로 stop
        - ps, ps -a 모두에서 없어짐

## docker 실습

- 사용 중인 docker 버전 확인(`$ docker version`)
    - docker desktop을 실행해야만 확인 가능
    
    ```
    Client:
     Cloud integration: v1.0.25
     Version:           20.10.16
     API version:       1.41
     Go version:        go1.17.10
     Git commit:        aa7e414
     Built:             Thu May 12 09:17:07 2022
     OS/Arch:           windows/amd64
     Context:           default
     Experimental:      true
    
    Server: Docker Desktop 4.9.0 (80466)
     Engine:
      Version:          20.10.16
      API version:      1.41 (minimum version 1.12)
      Go version:       go1.17.10
      Git commit:       f756502
      Built:            Thu May 12 09:15:42 2022
      OS/Arch:          linux/amd64
      Experimental:     false
     containerd:
      Version:          1.6.4
      GitCommit:        212e8b6fa2f44b9c21b2798135fc6fb7c53efc16
     runc:
      Version:          1.1.1
      GitCommit:        v1.1.1-0-g52de29d
     docker-init:
      Version:          0.19.0
      GitCommit:        de40ad0
    ```
    
- `docker run -dp 80:80 docker/getting-started` 진행 후 [localhost:80](http://localhost:80) 접속하면 getting started 화면 보기 가능
- Docker Desktop을 설치했다면 BuildKit 사용이 default이므로 따로 설정할 필요 없음
1. local machine에 실습할 dir 하나 생성
2. anaconda로 실습할 가상 환경 생성
3. 실습 dir에서 anaconda 가상 환경으로 진행
    - cmd에서는 grep 명령어가 사용되지 않으므로 bash로 변경했고 bash에서 anaconda 사용하는 방법은 참조한 페이지 참조
    
    ```
    pip3 install Flask
    pip3 freeze | grep Flask >> requriements.txt
    touch app.py
    ```
    
    ```python
    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def hello_world():
        return 'Hello, Docker!'
    ```
    
    - `python -m flask run` 으로 실행(python3는 안됐었음)
        - localhost:5000에서 확인 가능
4. image build
    - Dockerfile을 app.py와 동일한 dir에 만들기
    
    ```python
    # syntax=docker/dockerfile:1
    
    FROM python:3.8-slim-buster
    
    WORKDIR /app
    
    COPY requirements.txt requirements.txt
    RUN pip3 install -r requirements.txt
    
    COPY . .
    
    CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
    ```
    
    - `docker build --tag python-docker` 로 이름이 python-docker인 이미지 만들기
5. image run
    - `docker run -p 8000:5000 python-docker` 로 image를 container에서 run
        - [localhost](http://localhost):5000에 접속하여 결과 확인 가능
    - docker stop [container name]으로 container 중지하기
    - docker rm [container name]으로 container 삭제
6. DB 연결
    - mysql, mysql_config라는 volume을 만들어서 container를 삭제해도 유지되는 persistent DB, config를 담을 volume를 만듦
    - app과 DB가 소통하는 network를 따로 만듦(user-defined bridge network)
        
        ```python
        docker run --rm -d -v mysql:/var/lib/mysql \
          -v mysql_config:/etc/mysql -p 3306:3306 \
          --network mysqlnet \
          --name mysqldb \
          -e MYSQL_ROOT_PASSWORD=p@ssw0rd1 \
          mysql
        ```
        
        - -e: 환경 변수 설정
        - —rm: container 종료할 때 컨테이너 관련 리소스도 완전 삭제(일회성 container)
        - -d: background 실행
        - -v: volume 연결
        - -p: host machine에서 container에서 보고 있는 포트로 접속할 수 있게 함(설정하지 않는다면 [localhost:5000](http://localhost:5000) 등 host machine에서 확인 불가)
        - —network: 해당 network 사용
    - mysql 이 잘 설치되었는지 확인
        
        `winpty docker exec -ti mysqldb mysql -u root -p` 
        
        winptysms git bash에서 docker 내부에 접속할 때 사용하는 명령어
        
7. docker compose
    - 여러 개의 container를 한 번에 실행
    - 옵션 저장 가능
    - app.py가 있는 dir에 docker-compose.dev.yml 파일을 만들고 저장
        
        ```python
        version: '3.8'
        
        services:
         web:
          build:
           context: .
          ports:
          - 8000:5000
          volumes:
          - ./:/app
        
         mysqldb:
          image: mysql
          ports:
          - 3306:3306
          environment:
          - MYSQL_ROOT_PASSWORD=p@ssw0rd1
          volumes:
          - mysql:/var/lib/mysql
          - mysql_config:/etc/mysql
        
        volumes:
          mysql:
          mysql_config:
        ```
        
    - `docker-compose -f docker-compose.dev.yml up --build` 를 통해 image를 컴파일 하고 container 시작
8. 최종 working dir tree
    
    ```docker
    python-docker
    	|-- app.py
    	|-- docker-compose.dev.yml
    	|-- Dockerfile
    	|-- requirements.txt
    ```
    
9. docker github와 연동하여 CI/CD 진행
    - 앞선 실습 내용과 무관
    - SimpleWhaleDemo repo를 fork하여 진행
    - repo→ settings → secrets → actions → new secret 에 아래 두 secret 만듦
        - DOCKER_HUB_USERNAME, docker hub 아이디
        - DOCKER_HUB_ACCESS_TOKEN, docker hub에서 발급 받은 Personal Access Token
    - actions → enable action 한 후 새로운 action 만들기
        - fork한 repo에 이미 yml 파일이 2개 존재하는데 삭제 하고 진행함
        
        ```
        name: CI to Docker Hub
        
        # Controls when the workflow will run
        on:
          # master branch에 push 했을 때만 action 진행
          push:
            branches: [ "master" ]
        
          # Allows you to run this workflow manually from the Actions tab
          workflow_dispatch:
        
        # A workflow run is made up of one or more jobs that can run sequentially or in parallel
        jobs:
          # This workflow contains a single job called "build"
          build:
            # build를 가장 최신 Ubuntu에서 진행하도록 설정
            runs-on: ubuntu-latest
        
            # Steps represent a sequence of tasks that will be executed as part of the job
            steps:
        			# $GITHUB_WORKSPACE 아래 있는 repo 체크
              - name: Check Out Repo 
                uses: actions/checkout@v2
              
              # optimize 위해 cache 사용
              - name: Cache Docker layers
                uses: actions/cache@v2
                with:
                  path: /tmp/.buildx-cache
                  key: ${{ runner.os }}-buildx-${{ github.sha }}
                  restore-keys: |
                    ${{ runner.os }}-buildx-
        
              # PAT 이용 docker hub 로그인
              - name: Login to Docker Hub
                uses: docker/login-action@v1
                with:
                  username: ${{ secrets.DOCKER_HUB_USERNAME }}
                  password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}          
              
              # BuildKit 사용 Builder 설정
              - name: Set up Docker Buildx
                id: buildx
                uses: docker/setup-buildx-action@v1
                
              - name: Build and push
                id: docker_build
                uses: docker/build-push-action@v2
                with:
                  context: ./
                  file: ./Dockerfile
                  builder: ${{ steps.buildx.outputs.name }}
                  push: true
                  tags:  ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest
                  cache-from: type=local,src=/tmp/.buildx-cache
                  cache-to: type=local,dest=/tmp/.buildx-cache
              - name: Image digest
                run: echo ${{ steps.docker_build.outputs.digest }}
        ```
        
        - master에 push가 아니라 특정 tag를 달아서 github에 올릴 때만 Docker Hub에 push 하도록 할 수 있음(이렇게 위 코드 중 on 부분을 수정)
            
            ```
            on:
            	push:
            		tags:
            			-"v*.*.*"
            ```
            
            - `git tag -a v1.0.2` `git push origin v1.0.2` 를 이용하여 특정 tag를 붙여 push가 가능하고 이 때만 Docker hub에 push됨
    - CI to Docker Hub는 성공하지만 CI to GHCR은 모두 실패함
    - Docker Hub에는 `simplewhale` 이라는 이름으로 repo가 생김

## 참조

- [How to Get Started with Docker](https://youtu.be/iqqDU2crIEQ)
- [get started](https://docs.docker.com/get-started/)
- [Docker BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/)
- [git bash에서 anaconda 사용하기](https://reliablecho-programming.tistory.com/47)
- [Run you image as a container](https://docs.docker.com/language/python/run-containers/)
- [Use Volume & compose](https://docs.docker.com/language/python/develop/')
- [GitHub Docker 연결](https://docs.docker.com/language/python/configure-ci-cd/)
- [Deploy Docker containers on EC2](https://docs.docker.com/cloud/ecs-integration/)



혹시 오류나 더 좋은 방법을 발견하신다면 댓글로 알려주세요!
