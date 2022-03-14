---
title: "5 Memory Hierarchy"
excerpt: "access latency, cache memory, direct mapped, N-way associative, write-back, write-through, virtual memory"
date: 2022-03-14
categories:
  - ComputerArchitecture
tags:
  - Theory
  - ComputerArchitecture
  - MIPS
---

이 글은 컴퓨터구조론 카테고리의 다섯 번째 글입니다. 다른 글도 궁금하시다면 [컴퓨터 구조론 시작하기](https://dongwon18.github.io/computerarchitecture/Computer_Architecture_Start/)를 확인해주세요.

MIPS에서는 32bit 32 register를 사용한다.

## Access Latency

- Static Ram: 접근 속도가 0.5ns ~ 10ns 정도로 빠르다. 따라서 cache에는 주로 SRAM을 사용한다.
- Dynamic Ram: 접근 속도가 50ns ~ 70ns로 비교적 느리나 GB 단위의 크기이다. main memory에 주로 사용된다. Dynamic ram에서 데이터를 읽어 오는 속도는 대략 200 cycle 정도로 생각하면 된다.
- 그 외: flash storage, magnetic disk 로 갈수록 용량은 커지고 속도는 느려진다.

따라서 **hierarchy로 메모리를 관리**하게 된다.

## Memory hierarchy

Register(Processor 내부) → Cache → Main memory

cache에서 main memory사이에서는 **블록 단위**로 데이터를 옮긴다.  
(따라서 hit ratio를 생각할 때 필요한 데이터가 포함된 블록이 cache 내에 존재하는지 따진다)

cache 또한 L1, L2, L3로 나뉠 수 있는데 L1 cache, L2 cache는 core마다 존재하고 L3는 여러 core가 동시에 사용하며 L3 cache 보다 더 아래 단계에 external DRAM이 존재한다.

## Direct mapped

- Set이 없이 **한 주솟값에 대해 한 블록만 존재**한다.
- 주소 구성
tag, index, offset 순서로 이루어짐
    - offset: 블록 크기에 의해 결정됨(log<sub>2</sub> 블록당 byte)  
    예: 1 word/ block = log<sub>2</sub> 4 = 2bit  
    - Index: 캐시 내에 들어갈 수 있는 블록 수로 결정됨  
    예: 4KB cache, 4 words/block = 4KB / 16 B = 256 blocks in a cache  
    log<sub>2</sub> 256 = 8 bit  
    - tag:  남은 부분  
- index가 동일하지만 tag가 다른 블록이 존재할 수 있기 때문에 index를 확인하고 해당 index에 데이터가 있다면 tag 또한 원하는 데이터와 동일한지 확인한다.
- index가 동일한 블록이 이미 cache 내에 존재한다면 이를 replace한다.
- 블록의 크기가 커지면 spatial locality는 증가한다.  
주위에 위치한 데이터를 더 많이 cache에 올릴 수 있으므로
- 그러나 블록이 커지면 cache 내부에 존재할 수 있는 총 블록의 개수는 줄어들게 된다. 따라서 블록 miss rate이 더 높아지게 된다.  
원하는 블록이 cache 내부에 있을 확률이 줄어들기 때문
- 따라서 **block size와 miss rate은 trade off 관계**에 있다 볼 수 있다.

### Write Through

- cache에 있는 값을 **update할 때 main memory에 있는 값 또한 즉각적으로 갱신**한다.
- 그러나 write는 시간이 오래 걸리는 행위이기 때문에 **write buffer를 이용**해서 갱신할 값을 저장해놓고 buffer가 다 차면 이를 처리하는 식으로 진행된다.

### Write back

- cache에 있는 값만 **update를 하고 dirty bit을 high로 표시**하여 해당 값이 변경되었고 나중에 main memory에 업데이트 해야 함을 알린다.
- 이 또한 **write buffer를 이용**하여 진행된다.

### Store miss

위의 두 경우 모두 write할 값이 이미 cache에 있음을 가정한다. 그러나 cache에 해당 값이 존재하지 않을 수도 있다. 이 때는 두 가지 해결 방안이 존재한다.

1. No write allocate: memory에 있는 값만 업데이트하고 해당 값을 cache에 올리지 않는다.
2. Write allocate: load miss인 것처럼 해당 값을 cache에 올린 후 write through, write back에 따라 write를 진행한다. (앞으로 이 값을 쓸 것이라고 예상하고 cache에 올리는 것이다. 그러나 복사 붙여넣기 등 앞으로 쓰이지 않을 값을 모두 cache에 올리고 내리는 등의 비효율적인 측면이 발생한다.)

write 방식과 함께 조합해보면

- write through, no wirte allocate  
write hit: 캐시 업데이트, 메모리 업데이트  
write miss: 메모리 업데이트  
- write through, wire allocate  
write hit: 캐시 업데이트, 메모리 업데이트  
write miss: 블록 캐시에 fetch, 캐시 업데이트, 메모리 업데이트  
- write back, no write allocate  
write hit: 캐시 업데이트, dirty bit high로 세팅  
write miss: 메모리 업데이트  
dirty line replace: 메모리 업데이트  
- write back, write allocate  
write hit: 캐시 업데이트, dirty bit high로 세팅  
write miss: 블록 캐시에 fetch, 캐시 업데이트, dirty bit high로 세팅  
dirty line replace: 메모리 업데이트  

## N-way Set associative

하나의 주소에 대해 N개의 block이 존재할 수 있다.

- **2 way ~ 8 way를 주로 사용**하며 N이 커질수록 conflict miss는 줄어들지만 해당 블록을 찾기 위한 하드웨어 구성품의 수가 늘어난다.
- 64KB D-cache, 16 word block의 경우 SPEC2000에 따르면 4 way(8.3%), 8 way(8.1%)의 cache miss ratio가 별로 차이 나지 않으므로 적정한 선까지만 N을 늘리는 것이 좋다.
- set 내에 N개의 블록이 존재할 수 있다.
- set 내에 빈 자리가 없지만 새로운 블록을 fetch해야 할 경우 replacement policy를 적용하여 어떤 블록을 뺄지 결정한다.
    - Least Recently Used(LRU): 가장 이전에 사용된 데이터가 향후에도 사용될 가능성이 가장 낮다고 생각하고 변경한다. Overhead를 줄인 Pseudo LRU 방식도 존재한다.
    - Round Robin: 순서대로 replace하는 방식이다.
    - Random: 무작위로 선택하는 방식이다.
    
    LRU 또한 완벽한 방법이 아닐 뿐 아니라 이를 위해 너무 많은 자원이 필요하기 때문에 주로 round robin혹은 random 방식을 많이 사용한다.
    

## Fully associative

cache 내에 자리에 상관 없이 빈 곳이 있으면 블록을 fetch한다. 따라서 자유도가 매우 높다고 볼 수 있다. 어떤 블록이 어느 주소에 fetch 되어 있는지 모두 저장하고 있어야 하기 때문에 하드웨어 구성이 복잡해진다는 단점이 있다.

## Miss

- Compulsory miss(cold start miss)  
처음 시작할 때 데이터가 없을 때 발생하는 miss로 피할 수 없다.  
- Capacity miss  
cache size가 한정적이기 때문에 발생하는 miss로 이전에 대체된 블록이 다시 쓰이면서 발생하는 miss이다.  
- Conflict miss   
fully associative이 경우에는 발생하지 않는다.  
set이 차 있는 경우 그 set에 블록이 들어갈 수 없어 발생하는 miss이다.  

## Virtual Memory

- 프로그램마다 가상 메모리 공간을 가지고 있다.
- 디스크 공간까지 활용 가능하기 때문에 실제 메모리의 크기보다 클 수 있다.
- virtual page numer → physical page number로 변경되어 실제 주소로 변경된다. 이 때 page offset은 변경되지 않는다. 변경은 메모리 내에 저장되어 있는 page table에 저장된 쌍으로 결정된다.
- 만약 page table entry에 해당 페이지가 존재하면 hit, 존재하지 않으면 page fault가 발생하여 OS에서 처리하게 된다.(바꿀 page를 선택한다.) 이 때 dirty bit이 high면 디스크에 값을 저장한 후 교체한다.
- page fault를 줄이기 위해서는 pseudo LRU 방식이 주로 사용되며 write-back 방식을 사용한다.
- 그러나 page table entry는 상당히 큰 용량을 차지할 수 있다.  
  2<sup>20</sup> page가 가상 메모리에 있다고 가정하고 page entry 당 4 byte를 차지한다고 가정하면 총 4MB가 필요하다. 
이를 해결하기 위해 page의 크기를 늘리거나 multi-level page table이 사용된다.
- 또한 page table을 확인하기 위해서 메인 메모리에 계속 접근해야 해서 시간이 오래 걸리므로 Translation Look-asicde Buffer(TLB)를 추가로 사용하여 속도를 높이기도 한다.

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주세요!
