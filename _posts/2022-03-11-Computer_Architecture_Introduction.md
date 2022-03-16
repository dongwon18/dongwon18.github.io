---
title: "1 Computer Architecture Introduction"
date: 2022-03-11
categories:
  - ComputerArchitecture
tags:
  - Theory
  - ComputerArchitecture
  - MIPS
---

이 글은 컴퓨터구조론 카테고리의 첫 번째 글입니다. 다른 글도 궁금하시다면 [컴퓨터 구조론 시작하기](https://dongwon18.github.io/computerarchitecture/Computer_Architecture_Start/)를 확인해주세요.

## 8 great idea

- Design for Moore’s Law  
IC 칩 내 트랜지스터 개수가 2년마다 2배가 된다
- Use abstracton to simplify design
- Make the **common case faster(Amdahl’s Law)**
- Performance via **parallelism**
- Performance via **pipelineing**
- Performance via **prediction**
- **Hierarchy** memory
- **Dependability** via redundancy

## HW abstraction

- ISA: Instruction set architecture  
하드웨어를 소프트웨어 적으로 어떻게 표현했는가  
CPU Instruction  , x86, ARM
- ABI: Application Binary Interface  
ISA + OS  
x86 Windows vs. ARM iOS  
ABI 까지 동일해야 물리적으로 다른 system에서도 동일한 결과가 나올 수 있다.

## History

Chip-to-chip data communication → System-on-Chip → Chiplet

- Chip-to-chip: 1980 년대 IBM의 형태로 하나의 칩에 트랜지스터를 많이 넣을 수 없었다. chip 사이 데이터가 이동하기 위해서는 선을 통해서 이동해야 했기 때문에 느렸고 중간에 loss도 많았기 때문에 전력 소모도 컸다. 또한 값이 비싸고 reliability 또한 낮았다.
- System-on-chip: 요즘 대부분 사용되고 있는 형태로 chip의 수가 줄어들고 functionality를 집재하였다.
- Chiplet: 작은 chip이라는 뜻으로 AMD Ryzen은 이 형태이다. 가격이 비교적 저렴하고 chip 사이에 communication을 할 확률을 매우 낮게 설계를 하여 효율성을 증대시켰다. 또한 예전처럼 wire로 연결하는 것이 아닌 반도체로 칩을 연결한다.

## IC Chip cost

cost per die = (cost per wafer)/(dies per wafer * Yield)  
Yield = 1 /  ((1 +(Defects per area * Die area / 2))<sup>2</sup>)  

웨이퍼를 크게 자를수록(칩의 크기가 커질수록) defect가 많아진다.

## Performance

CPU Time = (CPU Clock Cycles) / (Clock Rate)  

성능을 향상시키려면 **clock cycle 수 줄이기, clock rate 높이기** 두 가지 방법이 있다. 그러나 이 두 가지는 trade off 관계이므로 하드웨어 설계 시 주요 고려 대상이다.

- Clock cycle  
Instruction Count * Cycles per Instruction  
Instruction Count: 프로그램을 어떻게 설계하는지에 따라 달라짐   
Cycle per Instruction: processor 설계에 따라 달라짐, instruction type에 따라 달라짐   

이 외에도 Benchmark 할 CPU를 정하고 이와 비교했을 때 어느 정도 성능을 가지는지 파악하는 방식도 존재한다. 예로 SPEC CPU2006 등이 있다.

이 글은 배운 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주세요!
