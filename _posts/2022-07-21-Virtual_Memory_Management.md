---
title: "7 Virtual Memory Management"
excerpt: "cost, HW component, SW component"
date: 2022-07-21
categories:
  - OS
tags:
  - Theory
  - OS
---

## Cost

Page fault rate = # of page fault / 총 길이

## HW component

- address translatio device
    - dedicated page table registers, cache memory, TLB
- bit vector
    - reference bit: 쓰이면 1로 설정, 주기적으로 0으로 초기화됨
    - update bit(=dirty bit): 수정되면 1(write back), 주기적으로 초기화 되지 않음

## SW component

### allocation strategies(how, how much)

- 각 process에 얼마나 할당할 것인가?
- fixed allocation, variable allocation
- process가 필요한 정도를 예측해야함

### Fetch strategies(when)

- 언제 page를 mem에 올릴 것인가?
- demand fetch, anticipatory fetch(예측하는데 kernel 개입 → overhead)

### Placement strategies(where)

- 어디에 처음 넣을 것인가?
- segmentation system에만 필요함
- first fit, best fit, worst fit, next fit

### Replacement strategies(who)

- 누구를 내보내고 새로운 걸 들일 것인가?
- fixed allocation: MIN, random, FIFO(많은 mem을 할당할수록 fault가 증가하는 Belady’s anomaly가 발생하기도 함), LRU(성능이 좋아서 주로 이 방식을 사용함, timestamp를 저장하지 않고 비슷한 성능 내는 방식인 stack 사용-중간에서도 뽑을 수 있음), additional reference-bits, LFU(LRU의 timestamp보다는 빠름, 그러나 최근에 사용된 page를 교체할 수도 있음), NUR, clock(초반에는 FIFO, 나중에는 LRU랑 비슷하게 동작), enhanced clock
- variable allocation: VMIN, Working set, page fault frequency

### Cleaning strategies(when)

- 언제 write back 할 것인가?
- demand cleaning(mem에서 page가 교체될 때), anticipatory(마찬 가지로 overhead로 인해 주로 demand 사용)

### Load control strategies(how many)

- 얼마나 load 할 것인가?
- multiprogramming 측면에서 생각
- underloaded → plateau → overloaded(fault가 너무 많이 발생해서 thrashing 발생)  

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주신다면 감사하겠습니다.
