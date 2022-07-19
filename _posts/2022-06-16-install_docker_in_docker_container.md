---
title: "docker container에 docker 설치"
excerpt: "도커 컨테이너 접근, bash로 docker 설치"
date: 2022-06-16
categories:
  - DevEn
tags:
  - DevEn
  - docker
---

안녕하세요. docker container에 docker를 설치하는 방법을 기록해두려고 합니다.

1. power shell 관리자 권한으로 실행
2. docker desktop 실행
3. 아래 명령어 입력
    
    ```
    $ docker exec -it [container id] /bin/bash
    # curl -fsSL https://get.docker.com -o get-docker.sh
    # sh get-docker.shl
    ```
    
    - `You may press Ctrl+C now to abort this script.
    + sleep 20`
    에서 멈춰있는데 계속 기다리면 됨
    
    결과
    
    ```
    # Executing docker install script, commit: b2e29ef7a9a89840d2333637f7d1900a83e7153f
    
    WSL DETECTED: We recommend using Docker Desktop for Windows.
    Please get Docker Desktop from https://www.docker.com/products/docker-desktop
    
    You may press Ctrl+C now to abort this script.
    + sleep 20
    + sh -c apt-get update -qq >/dev/null
    + sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
    debconf: delaying package configuration, since apt-utils is not installed
    + sh -c mkdir -p /etc/apt/keyrings && chmod -R 0755 /etc/apt/keyrings
    + sh -c curl -fsSL "https://download.docker.com/linux/debian/gpg" | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
    + sh -c chmod a+r /etc/apt/keyrings/docker.gpg
    + sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bullseye stable" > /etc/apt/sources.list.d/docker.list
    + sh -c apt-get update -qq >/dev/null
    + sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-scan-plugin >/dev/null
    debconf: delaying package configuration, since apt-utils is not installed
    + version_gte 20.10
    + [ -z  ]
    + return 0
    + sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce-rootless-extras >/dev/null
    debconf: delaying package configuration, since apt-utils is not installed
    
    ================================================================================
    
    To run Docker as a non-privileged user, consider setting up the
    Docker daemon in rootless mode for your user:
    
        dockerd-rootless-setuptool.sh install
    
    Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.
    
    To run the Docker daemon as a fully privileged service, but granting non-root
    users access, refer to https://docs.docker.com/go/daemon-access/
    
    WARNING: Access to the remote API on a privileged Docker daemon is equivalent
             to root access on the host. Refer to the 'Docker daemon attack surface'
             documentation for details: https://docs.docker.com/go/attack-surface/
    
    ================================================================================
    ```
    
- `docker --version`실행 결과 docker 버전이 잘 출력됩니다.
- container bash에서 power shell로 돌아오려면 `exit`을 사용합니다
