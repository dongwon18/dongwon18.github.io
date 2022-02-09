---
title: "2 Link Layer(Layer 2)"
excerpt: "Error Detection Code, Multiple Access Protocol, LAN, Multi Protocol Label Switching, 구글 접속 예제"
date: 2022-02-09
categories:
  - ComputerNetwork
tags:
  - Theory
  - ComputerNetwork
---

안녕하세요.   
이 글은 Computer Networks 시리즈의 두 번째 글입니다. 다른 글도 궁금하시다면 [Computer Networks 시작 page](https://dongwon18.github.io/computernetwork/Computer_Network_start/)를 참고해주세요.

- Link layer는 Internet protocol stack의 하위 2번째 layer(laye2)로 물리적으로 인접한(one hop neighbor) node가 datagram을 주고 받을 수 있도록 합니다.
- datagram에 header, error detection code를 덧붙여 frame으로 만듭니다.
- header에서는 MAC 주소(LAN 카드에 부여된 주소)가 사용됩니다.
- 또한 간단한 error detection, correction, flow control, duplex를 담당합니다.
- link layer는 인접한 노드 간 reliable delivery는 진행하지 못합니다. 따라서 이 부분은 transport layer에서 진행됩니다.
- 키워드: Ethernet, LAN, switch, MAC address, ARP, MPLS

## 용어 정리

- node: host, router
- link: 인접한 node를 communication path를 통해 연결하는 channel  
wired, wireless, LAN 등

## Error Detection Code

- Error detection & correction(EDC)을 진행한다고 해서 100% 에러가 감지되거나 고쳐지는 것은 아니다.
- EDC코드가 길수록 에러 검출 및 복원 능력이 뛰어나다.
- EDC방법에는 여러가지가 있다.
    - Parity Checking
        - 컴퓨터의 데이터는 0 혹은 1로 이뤄져 있으므로 해당 데이터에 1이 짝수 개(even parity check) 혹은 홀수 개(odd parity check) 남도록 비트 하나를 정하는 것이다. 2차원으로도 진행될 수 있다.
        - 예를 들어 데이터가 1011이고 even parity를 사용한다면 데이터에 1이 3개이므로 1개의 1이 더 있어야 1이 짝수 개가 되므로 10111처럼 1을 앞 또는 뒤에 더한다.
    - Internet Checksum
        - 전송할 데이터가 총 4개라면 4개를 모두 더한 값의 보수를 checksum으로 보낸다.
        - 예를 들어 1000, 1001, 1011, 1010의 데이터를 보낸다고 가정하자. 이 때 4번째 자리에서 올림이 있다면 이 수를 가장 아랫자리에 더해준다.  
        1000 + 1001 = 10001 → 0001 + 1 = 0010  
        0010 + 1011 = 1101  
        1101 + 1010 = 10111 → 0111 + 1 = 1000  
        최종적으로 데이터를 모두 더하면 1000이 되고 이 수의 보수인 0111이 checksum이 된다.
        - 구한 checksum은 데이터와 함께 보내진다.(위의 예시에서는 데이터 4개 + checksum 1개 총 5개의 데이터가 보내진다.)
        - 수신 측에서 checksum을 포함한 데이터를 모두 더한 값을 1’s complement(1의 보수)취했을 때 0이 되면 에러가 없는 것으로 판단하고 0이 아니라면 에러라고 판단한다.
    - Cyclic Redundancy Check(CRC)
        - Polynomial code라고도 불리며 다른 방식보다 에러 검출 성능이 뛰어나다.
        - data bit 뒤에 CRC bit을 덧붙여 송신한다. CRC bit는 이진 데이터(D)를 송수신 측이 미리 정의한 패턴(G)으로 나눈 것의 나머지이다. 이 때 일반적인 산수 방식으로 더하기, 빼기를 진행하지 않고 모두 XOR로 진행한다.
        - 예를 들어 D = 101110000, G = 1001일 때 몫= 101011, 나머지(R)=011이고 최종적으로 데이터에 나머지를 덧붙인 101110000011이 CRC를 접목한 결과가 된다.

## Multiple Access Protocol

- link에는 point to point, broadcast 2가지 방식이 있다. 이 중 broadcast를 진행하는 다양한 방식이 존재한다.
- 한 번에 2개 이상의 송수신이 동일한 채널 내에서 진행되는 경우 만약 하나의 node가 동시에 2개 이상의 신호를 받게 되면 **collision**이 발생한다.
- multiple access protocol은 한 채널을 여러 node가 사용하고 있을 때 어떻게 채널을 나눌지 결정하는 알고리즘이다.

### Channel partitioning

- 이상적인 방식  
하나의 노드가 송신을 할 때는 R bps로 데이터를 송신할 수 있다고 가정하자. 여러 노드가 동일하게 채널을 나눠 사용한다면 n개의 노드가 사용할 때는 n/R bps로 균등하게 채널을 나눠 가질 수 있다. 이 방식은 간단하고 직관적이지만 단점도 존재한다.
- 사용하지 않는 자원이 낭비되는 것이 대표적인 단점이다. 즉 한 노드가 사용하지 않는다고 다른 노드에게 빌려주는 등의 형식이 불가능하다.
- 채널을 나누는 방식에는 크게 3가지가 있다.

#### TDMA  
시간을 나누는 방법, 각 노드가 라운드마다 전송할 내용이 있다면 전송하는 방식이다. slot의 길이는 동일하게 설정한다. 사용하지 않는 slot은 낭비되게 된다.

#### FDMA  
채널의 주파수 대역을 나누는 방식이다. 각 station마다 정해진 주파수 대역을 가진다. 이 방식 또한 해당 영역을 할당 받은 노드가 통신을 하지 않는다면 해당 대역이 낭비된다. 

#### CDMA  
코드를 할당하는 방식이다. 송수신 측이 사전에 동의한 코드를 전송 받은 데이터에 곱해 자신에게 온 데이터를 알아내는 방식이다. 채널의 시간이나 주파수를 나누는 것이 아니기 때문에 앞에서 말한 TDMA, FDMA 와는 다른 방식이라고 볼 수 있다.   
예를 들어 전체 채널에 1 -1 3 -1 1 3 -1 -1과 같은 값이 들어왔다 하자(0, 1이 아닌 3이 존재하는 것은 여러 노드가 1을 동시에 보내 1이 3개 중첩되었기 때문이다. 1은 물리적으로 high 를 나타내고 수신된 high 신호의 진폭이 1인 high보다 더 크게 나타난다.) A라는 노드는 -1 -1 -1 1 1 -1 1 1이라는 코드를 가지고 있을 때 각각의 코드를 해당 자릿수에 곱하면 -8이 나온다. 총 데이터의 길이인 8로 나누면 -1이 나오고 이 값은 0이라는 것을 알 수 있다.(송수신할 때 소비되는 전력의 측면에서 0, 1V로 0과 1을 나타내는 것보다 -0.5V, 0.5V로 나타내는 것이 효율적이기 때문에 이와 같이 -1을 0으로 나타낸다.) 

### Random Access Protocol

앞서 말했던 채널을 나눠 송수신하는 방식은 낭비의 단점이 있다. 따라서 실제로는 Random access protocol을 가장 많이 사용한다.

- 채널을 랜덤하게 **독차지** 한다. 즉 bandwidth를 한 node가 모두 사용한다.
- 노드 간 우선 순위가 존재하지 않는다.
- 만약 동시에 2개 이상의 노드가 송신을 하면 **collision**이 발생한다.
- Random Access MAC protocol에서는 어떻게 collision을 **감지**할지, 어떻게 **해결**할지(recovery)를 정한다.
- ALOHA, CSMA, CSMA/CD, CSMA/CA 등의 protocol이 있다.

#### Slotted ALOHA

- frame의 사이즈가 동일하다.
- time slot 또한 길이가 동일하다(frame 한 개를 전송할 수 있는 시간으로 맞춤)
- time slot이 시작할 때만 frame 전송을 시작한다.
- time slot을 맞춰야 하기 때문에 노드들은 동기화 되어 있다.
- 한 번에 2개 이상의 노드가 송신을 하려 하면 collision이 발생하고 모든 노드가 collision이 발생했음을 안다.
- collision이 발생하면 일정 확률로 재전송하고 성공할 때까지 반복한다.
- 장점  
active node가 하나밖에 없다면 그 node가 계속 송신할 수 있다.  
간단하다.  
중앙 제어가 필요 없고 한 slot내에 존재하는 노드만 동기화하면 된다.
- 단점  
collision이 발생할 수 있고 발생한다면 slot이 낭비된다.  
빈 slot은 낭비된다.  
clock 동기화가 필요하다.
- efficiency

$$
lim_(N→∞) (Np(1-p)^(N-1)) = 1/e = 0.37
$$

#### (unslotted) ALOHA

- Slotted ALOHA보다도 단순하다.
- 동기화가 필요 없다.
- slot이 나눠져 있지 않다.
- 송신할 게 있다면 바로 송신한다.
- 해당 노드가 송신하기 전에 송신하고 있던 노드가 없어야 하고 해당 노드가 송신이 끝나기 전에 송신하는 노드가 없어야 하기 때문에 충돌 확률이 더욱 높아진다.
- efficiency
    
    $$
    lim_(N→∞) (Np(1-p)^2(N-1)) = 1/2e = 0.18
    $$
    

#### CSMA(Carrier Sense Multiple Access)

- 보내기 전 채널이 idle, 비었는지 확인한다.
- 만약 비었다면 모든 frame을 다 전송하고 비지 않았다면 기다린다.
- 만약 collision이 발생한다면 packet를 전송한 시간 전부가 낭비되게 된다.

#### CSMA/CD(Collision Detection)

- Ethernet에서 사용하는 방식이다.(wireless의 경우 신호가 매우 미약하기 때문에 collision을 감지하기 매우 힘들다. 따라서 wireless의 경우에는 CSMA/CA 방식을 주로 사용한다.)
- 만약 채널이 비어 있다면 송신을 시작한다  
→ 만약 전송 중 collision(다른 node가 송신)을 감지했다면 즉시 송신을 멈추고 jam signal을 보낸다. 그리고 binary(exponential) backoff 모드에 돌입한다.
→ 만약 전송 중 collision을 감지하지 않았다면 전송을 성공적으로 끝낸다.
- 만약 비어있지 않다면 빌 때까지 기다린다.
- binary backoff는 지수적으로 기다리는 시간이 늘어나는 모드이다. 예를 들어 처음에는 {0, 1}중 랜덤으로 택하여 그 시간만큼 기다렸고 다시 시도 했을 때 여전히 채널이 busy라면 {0, 1, 2, 3}$2^2$중 랜덤으로 선택하여 기다리는 식이다.
- 송신하려는 node의 개수가 증가함에 따라 collision 발생 빈도가 높아지고 efficiency는 감소한다.그럼에도 불구하고 ALOHA보다는 좋은 성능을 가진다.
- ALOHA보다 간단하고 저렴하며 성능 또한 좋으므로 훨씬 많이 사용된다.

#### CSMA/CA(Collision Avoidance)

- wireless 통신에서 사용된다.
- collision을 감지하고 대비하는 것이 아니라 collision 상황을 피하는 것이 우선이다.  
wireless의 경우 신호가 매우 미약하기 때문에 collision이 발생했는지 여부 자체를 파악하는 게 힘들기 때문에 collision에 대응한다는 것 자체가 불가능하다.
- 송신 전에 미리 채널이 비었는지 확인한 후 송신하는 방식을 사용한다.
- 그럼에도 불구하고 완벽하게 collision avoid를 보장할 수는 없다.
- 송신: DIFS 후 RTS 보냄 → CTS 받으면 SIFS 후에 데이터 보냄
- 수신: RTS 받은 후 SIFS만큼 기다림 → CTS 송신 → data 받으면 SIFS 후 ACK 보냄

### Taking Turns

- channel partitioning은 동일하게 채널을 나눠 받지만 쓰지 않는 것은 낭비되고 active node가 1개 일 때도 채널을 모두 사용할 수 없다는 단점이 있다. Random Access는 active node 1개가 채널을 모두 사용할 수는 있지만 collision overhead가 존재한다.
- 이 두 가지의 절충안이 taking turns 이다.

#### polling

```jsx
master
|--slave1
|--slave2
|--slave3
```

의 형식으로 라인이 정립된다.   

- slave들이 본인의 순서에 맞춰(순서는 master가 지정한다.) 데이터를 송신한다.
- polling overhead, single point of failure(master)의 단점이 있다.

#### token passing

```jsx
slave1 ---- slave2
  |           |
  |           |
  |           |
slave3 ---- slave4
```

의 형식으로 라인이 정립된다.

- master가 따로 없고 token을 받으면 본인의 차례라고 인식하고 data를 보낸다.
- token overhead, single point of failure(token)의 문제가 발생한다.

## LAN

### MAC & ARP

#### MAC

- 48 bit 하드웨어 주소이다.
- LAN card에 할당되어 있다.
- IEEE가 MAC 주소를 LAN card를 생산하는 회사에 파는 형식으로 관리한다.  
IP address의 경우 subnet에 의해 결정되기 때문에 MAC주소처럼 HW를 옮긴다고 변형되지 않는다.

#### ARP

- 실제로 통신을 하기 위해 사용하는 주소는 IP 주소이다.
- 따라서 MAC address와 IP 주소를 mapping 해주는 것이 필요하고 이를 **ARP table**이라고 부른다.
- 같은 LAN에 있을 때(router를 거치지 않아도 될 때)  
A가 같은 LAN에 존재하는 B의 MAC 주소를 알지 못할 때
    1. A가 ARP query를 broadcast한다. 이 때 B의 IP address를 알고 있다고 가정한다. 
    (현실적인 가정이라 생각한다. URL은 결국 IP address와 1대 1 mapping된다. URL은 알아야 해당 홈페이지에 접속을 할 수 있으니 합리적이다.)
    broadcast 시에는 src mac address, ip address, dst mac address, ip address를 담아 query를 보내는데 dst mac address는 48자리 모두 1로 설정한다.
    2. 같은 LAN 내 모든 노드가 broadcast query를 받는다.
    3. B는 A에게 본인의 MAC address를 보낸다. 이 때는 1:1 unicast 전송을 한다.
    4. A의 ARP table에 B의 MAC, IP 주소가 저장된다.
    5. Time To Live 시간이 끝나면 해당 데이터는 오래된 데이터 혹은 사용하지 않는 데이터로 간주되어 삭제된다.(주로 20분이다)
- plug and play이다.

### Ethernet

- 가장 많이 사용되는 wired LAN 기술이다.
- 어댑터가 FDDI, ATM과 같은 다른 기술에 비교할 때 아주 저렴하다.
- data rate이 높다.
- Ethernet frame  
preamble |  dst addr | src addr | type | data | CRC  
preamble: 10101010을 반복하는 7byte동안 반복하고 마지막 byte는 10101011로 동기화용 마지막 코드임을 나타낸다.  
src, dst addr: MAC addr  
type: layer 3의 protocol 종류, 주로 IP이다.
- Ethernet은 unreliable, connectionless이다. 따라서 TCP와 같은 더 높은 layer의 protocol에서 이를 다뤄줘야 한다.

### Switch

- link layer에서 사용되는 device이다.
- ethernet frame을 store하고 forward할 수 있다.  
즉 dst addr를 보고 어디로 forward해야할 지 정한다. 이 때 CSMA/CD(Ethernet switch의 경우)를 사용한다.
- plug-and-play로 진행된다.
- dst가 어디에 있는지 self learning한다.
- MAC addr, interface, TTL이 포함된 테이블을 사용한다.  
만약 테이블에 dst addr가 존재하지 않는다면 본인(arriving interface)을 제외한 모든 node에게 forward한다.    
→ 이 때 arriving interface의 정보가 테이블에 없다면 이 또한 등록한다.
→ 본인이라면 response를 보낸다.  
→ switch table에 response를 보낸 node의 interface 번호와 MAC addr를 저장한다.

### VLAN

- 하나의 swith를 마치 서로 다른 switch인 것처럼 분리하여 사용하는 방식이다.
- traffic isolation, dynamic membership, forwarding betwen VLANS 가 가능해진다.
- 반대로 물리적으로 다른 switch를 하나처럼 사용하는 것도 가능하다. 이 때는 trunk port를 할당하여 두 switch를 연결하여 하나처럼 사용할 수 있도록 한다. 즉 port 개수를 늘리기 수월하다.

## Multi Protocol Label Switching

- IP 주소 대신 정해진 길이의 label의 사용하여 forwarding하는 기술이다. 마치 Virtual Circuit처럼 동작한다.
- IP path의 경우 **dst address**에 따라 하나의 path만 결정된다.
- MPLS의 경우 **src, dst**에 대해 다양한 path를 사용할 수 있다. 따라서 failure에서 back up 하는 시간이 짧다.
- router마다 in label, out label, destination, out interface로 path를 표시한다.
- MPLS는 router에서 이뤄지지만 IP 주소와 다른 네트워크 layer 들의 주소를 사용하기 때문에 완벽히 layer 3은 아니다. 그러나 multi hop을 하기 때문에 single link만 다루는 layer 2도 아니기 때문에 **layer 2.5**에 해당하는 기술로 분류된다.

## 구글 접속 예제

1. DHCP로 노트북의 IP address를 할당 받는다. 이 때 first hop router IP address, name & IP addess of DNS server 또한 받는다.(UDP로 진행)
2. 노트북에서 URL을 입력한다.
3. DNS server에서 해당 URL에 맞는 IP 주소를 받는다.(UDP로 진행)
    1. router에 frame을 보내기 위해서는 MAC address가 필요하다. 따라서 ARP Query를 진행한다.
    2. first hop router의 IP address를 알고 있기 때문에 ARP Query로 first hop router의 MAC address를 알아낸다.
    3. DNS query를 전송하여 DNS server에서 reply를 받는다.
4. HTTP request를 보내기 위해 client에서 TCP socket을 열어야 한다.
    1. 3-way handshake를 이용하여 연결을 진행한다.
    2. 연결을 끊을 때는 4-way handshake를 이용한다.
5. HTTP request를 TCP socket에 넣고 web server로 route 한다.
6. HTTP reply(index.html)를 받아 브라우저에서 표현한다.

이 글은 배운 내용을 정리한 글으로 오류가 있을 수 있습니다. 오류를 발견하셨다면 댓글로 알려주시면 감사하겠습니다.
