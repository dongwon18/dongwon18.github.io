---
title: "1 Computer Networks Introduction"
excerpt: "defi, packet switching vs circuit switching, delay, internet protocol stack, OSI layer"
date: 2022-01-25
categories:
  - ComputerNetwork
tags:
  - Theory
  - ComputerNetwork
---

안녕하세요.   
이 글은 Computer Networks 시리즈의 첫 번째 글입니다. 다른 글도 궁금하시다면 [Computer Networks 시작 page](https://dongwon18.github.io/computernetwork/Computer_Network_start/)를 참고해주세요.

## Introduction

- Internet은 전세계를 연결하는 wide area network이다.
- 서로 소통하기 위해 지켜야 할 protocol이 존재한다.
- network edge에는 client와 server가 host로서 존재한다.

## Packet Switching(layer2)

- packet 단위로 나눠서 송수신 진행. 즉 정해진 사이즈의 데이터를 주고 받음
- path 공유
- 고정된 path가 아닌 그 때 그 때 다른 path
- 고정된 path가 아니기 때문에 path가 낭비되지 않음
- queuing delay & loss 발생
path를 공유하기 때문에 독점적으로 사용하는 path가 없으므로 해당 path가 사용중이라면 delay가 발생한다.
- packet을 동시에 여러 개를 보내고 router에서 순서를 맞춤(store & forward)
- router에서 순서를 맞추는 등의 행위를 하기 때문에 queuing delay, packet loss 발생 가능
- 모든 packet이 와야 다음 layer로 이동하게 됨

## Circuit Switching

- packet을 나누지 않음
- dedicated path
- 한번 path를 setup 하고 나면 그 path를 그대로 사용  
packet switching과 달리 set up time이 필요하다.
- loss & delay가 없어서 높은 신뢰도가 필요한 통신에서 사용  
한 path를 점유하고 있기 때문에 store할 필요 없이 바로 송신한다. 따라서 store & forward 하지 않는다.
- 해당 path를 사용하지 않는다면 낭비됨
- 동시에 여러 노드에서 송신을 원할 경우 setup time이 길어지며 각각의 노드에 대해 dedicated path를 할당하고 해제하는 방식으로 동작

## Network of Networks

- Internet Service Provider(ISP)  
인터넷 접속 서비스를 제공하는 회사로 예로 KT, SK브로드밴드, LG U+ 등 흔히 알고 있는 통신사가 있다.
- ISP를 모두 연결하기 위해서는 $nC_2$  만큼의 연결이 필요하다. 즉 $n^2$에 비례하는 연결이 필요하게 된다. 따라서 **global ISP**를 두어 이를 통해 소통을 하도록 하여 연결 비용을 줄인다.
- 실제로 global ISP 하나가 한 통신사로 이뤄지는 경우가 많다. 따라서 다른 통신사를 사용하고 있는 지인과 소통을 하기 위해서는 global ISP 또한 연결되어 있어야 한다. 이러한 연결을 담당하는  connection node를 **Internet Exchange Point(IXP)**라고 부른다.

## Packet Delay

총 4가지의 delay가 존재한다.

<p align = "center">
  <img src="![Untitled](https://user-images.githubusercontent.com/74483608/150898163-f128d58f-7cdd-4bae-8ffe-760d8ac12e56.png)" alt="packet delay"> <br />
  four kinds of packet delay (출처: computer networking a top down approach 발행 ppt)
</p>

- processing  
error bit check, output link 등 전 packet header를 처리하는데 걸리는 시간  
일반적으로 10^-3 sec 정도로 짧기 때문에 delay를 계산할 때 무시할 때도 많음
- queuing  
queue에서 자신의 차례를 기다리는 시간  
network의 부하에 따라 결정되기 때문에 가변적이고 network delay에 가장 많은 영향을 줌  
가변적이기 때문에 주로 컴퓨터 네트워크 delay 문제를 다룰 때는 무시함
- transmission  
모든 bit을 wire에 전송되도록 하는 시간  
packet size / bandwidth임
- propagation  
데이터가 wire를 타고 이동하는데 걸리는 시간  
link distance / propagation speed 임  
전파율이 좋은 wire를 사용한다면 해당 delay가 줄어듦

## Circuit switching vs Packet switching Delay

processing time, queuing delay 무시, 모든 노드 간의 거리 및 propagation speed가 동일하다고 가정할 때 다음과 같다.

- Circuit Switching
    - setup time이 존재
    - total delay = setup time + transmission time + last bit propagation time  
    = setup time + message size/link bandwidth + link distance * hop / signal propagation speed
    - 만약 노드 간의 거리, propagation speed가 다르다면 각 구간마다 propagation time을 구해 더해주면 된다.
    - packet header를 다루는 processing time은 존재하지 않는다. packet 단위가 아니기 때문에 packet header가 존재하지 않을 뿐 아니라 path가 이미 setup time에 설정되었기 때문에 정해진 path로 송신만 하면 되기 때문이다.
- Packet Switching
    - setup time 존재 안함
    - 마지막 packet이 travel 하는 기준
    - total delay = transmission time + last packet travel time  
    = message size/link bandwidth + (last bit propagation time + store and forward time * intermediate node)
    - 이 때 store and forward time은 packet transmission time과 동일하다.
    - 만약 processing time을 고려한다면 intermediate node마다 processing time을 고려하기 때문에 last packet travel time = last bit propagation time + store and forward time * intermediate node + processing time * intermediate node가 된다.

## Internet Protocol Stack

아래부터 위로 physical → link → network → transport → application의 구조이다. 송수신하는 예를 들자면  
송신측: application → transport → network → link → physical 순으로 전송하고자 하는 데이터가 layer 하부로 이동한다.  
수신측: physical → link → network → transport → application 순으로 layer 상부로 올라와 확인할 수 있게 된다.

- physical: 가장 하부에 위치하며 네트워크 하드웨어 전송기술을 다룬다. 모뎀, 허브, 리피터 등이 해당한다.
- link: 네크워크 상 주변노드와 소통한다. 관련 용어로 ethernet, ethernet frame, MAC address, Switch, flow control, error detection & correction 등이 있다.
- network: host간 logical communication을 담당한다. routing & forwarding을 진행하며 IP datagram, router, forwarding, broadcast, multicast 등이 관련 용어이다.
- transport: 서로 다른 host 내에 존재하는 application process가 logical communication을 하도록 한다. TCP, UDP, segment, multiplexing, congestion control 등이 관련 용어이다.
- application: 인터넷 네트워크를 사용한다. 관련 용어는 HTTP, client-server, P2P 등이다.

## ISO/OSI Reference Model

앞서 다룬 Internet Protocol Stack과 비슷하지만 추가된 layer가 2개 존재하여 총 7개의 layer이다.  
physical → link → network → transport → session → presentation → application

- session: synchronize, checkpoint
- presentation: encryption, machine-specification

해당 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 만약 오류를 발견하셨다면 댓글로 알려주세요!
