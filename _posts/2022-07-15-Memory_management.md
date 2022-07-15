---
title: "6 Memory Management"
excerpt: "Contiguous/Discontiguous Allocation, ,FPM, VPM, Paging, Segmentation, Virtual Memory"
date: 2022-07-12
categories:
  - OS
tags:
  - Theory
  - OS
---
## Contiguous Allocation

- process의 context가 연속된 메모리를 할당받음
- 본인에게 할당된 메모리 주소 외 공간을 접근하려고 했을 때 막아야 함 → limit register 값 체크

### Uniprogramming

- process 1개인 상태로 모든 memory를 해당 process가 사용하면 됨
- program size > memory- size인 경우 overlay structure를 사용함(코드를 나눠서 부분만 mem에 올림)
- mem 쓸 때는 CPU가 놀고 그 반대이기도 하니 효율성 떨어짐 → multiprogramming 사용
- MS-DOS

### FPM

- mem을 고정된 크기로 나눔
- 하나의 process가 하나의 partition을 받도록 함
- 간단하고 overhead 적음
- 자원 관리 능력 떨어짐
- internal, external fragmentation
    - internal: 필요한 메모리 공간보다 할당 받은 공간이 작을 때 발생
    - external: 전체적으로 메모리 공간은 충분하지만 연속적이지 않아 할당받지 못할 때 발생

### VPM

- mem을 필요한 만큼 나눠줌 → internal framentation 존재 안함
- 어느 자리에 들어갈 지 정하는 placement strategies 필요
    - first-fit: 처음부터 찾기 시작해서 자리 발생하면 바로 들어감, 간단하고 부하 적음
    - best-fit: 모든 곳을 탐색하여 가장 적게 남는 자리 들어감, search time 오래 걸림, 작은 size partition이 많이 남을 수 있음 → external fragmentation
    - wortst-fit: 모든 곳을 탐색하여 가장 많이 남는 자리 들어감, search time 오래 걸림, 작은 size partition이 남는 문제는 해결
    - next-fit: 이전에 탐색했던 다음 자리부터 탐색해 들어갈 수 있는 자리 있음 바로 들어감, memory partition을 고르게 사용 가능, 부하 적음
- coalescing holes: 연속하는 곳에 빈 자리 있으면 큰 덩이로 합침
- storage compaction: 한 쪽으로 다 몰아 저장, 실행 시간 오래 걸림, 거의 안 함

## Discontiguous Allocation

- physical memory가 연속적이지 않아도 됨

### Paging

- page table이 존재하여 해당 page가 물리적 mem에는 어디 존재하는지 저장하고 있음
- 물리적 mem을 고정된 크기로 나눠 frame으로 만듦
- 프로그램의 mem을 고정된 크기로 나눠 page로 만듦
- address mapping, dedicated CPU register, TLB 등으로 구현됨,

### Segmentation

- 가변 크기로 할당함
- base, limit 체크해야 함

## Virtual memory

- 모든 logical mem이 mem에 올라오지 않아도 프로그램을 실행할 수 있도록 하는 개념
- program을 나눔
- 실행에 필요한 부분만 메모리에 올림
- 장점
    - 프로그래머가 메모리 사이즈 생각할 필요 없이 코딩 가능
    - multiprogramming 가능
    - 일부만 메모리에 적재되어 있으므로 swap 시간이 적게 듦
- 단점
    - address mapping overhead
    - page fault handling overhead
    - realtime, embedded에서는 사용에 유의해야 함

### Paging(Demand paging)

- program을 동일한 사이즈로 나눔
- 단순하고 효율적임
- 대부분의 OS에서 사용됨(segmentation과 혼합된 형태로)
- sharing, protection의 측면에서는 복잡해짐
    - sharing 할 때 base address가 다르므로 어느 걸 기준으로 계산해야 할지 애매함
        - copy-on-write: 다른 process에 의해 수정되는 page를 복사 해서 따로 쓰게 함
- Page Map Table에 저장됨(v=(page number, displacement)
    - PCB 내에 존재
    - physical address 구한다 → page fault면 context switching 아니라면 그냥 사용

### Segmentation

- 서로 다른 크기의 logical block으로 나눔

### Hybrid paging / segmentation

## Address Mapping

- direct mapping
    - 단점:
        - PCB보기 위해 mem 접근(접근 속도 100ns) + 실제 필요한 mem 접근 → 총 2번 접근으로 느림
        - PMT size가 거대함
        
        → TLB 사용, dedicated register, cache mem이용
        
- associative mapping(TLB)
    - parallel search 가능
    - overhead 적고 빠름(접근 속도 20ns)
    - HW가 비싸짐
- Hybrid mapping
    - full PMT를 mem에 저장
    - PMT의 일부를 TLB에 저장하여 사용(자주 사용하는 부분)

### Page table structure

- hierarchical paging
    - page table을 page한 table을 사용 → PMT 사이즈 문제 해결 가능
- hashed page table
- inverted page table
    - 모든 process 들이 하나의 page table을 사용함
    - page table 의 사이즈 줄어듦
    - 단점: table search 시간 증가 → hash table, TLB 이용으로 해결

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주신다면 감사하겠습니다.
