---
title: "3 Process Scheduling"
excerpt: "scheduling goal, performance 척도, sheduling 기법"
date: 2022-06-24
categories:
  - OS
tags:
  - Theory
  - OS
---


## Scheduling Goal

- 각 prcess를 케어하면서 system performance 향상
- multi process 환경에서 필요함(single Process는 필요 없음)
- 얼마나 효율적인지는 performance로 측정함
- time share, space share가 고려 대상임

## Performance 척도

- response time: 실제로 CPU 사용시간
- fairness
- throughput
- resource utilization
- predictability

## Scheduling 기법

### non-preemptive

- FCFS
    - arrival time이 우선인 순으로
    - 장점: 간단하다, resource utilization이 높다
    - 단점: starvation(convoy effect), priority reversal, response time 증가
- Short Process Next
    - runtime이 짧은 걸 기준으로 한다.
    - 장점: 평균 waiting time이 가장 적은 방식, response time 짧음
    - 단점: run time이 긴 건 starvation, run time 예측(over head 발생)

### preemptive

- Round Robin
    - arrival time이 우선인 순으로, 모든 Process가 CPU 사용 시간을 동일하게 가짐
    - 장점: FCFS의 starvation을 해결 가능,
    - 단점: time slice가 너무 길면 FCFS와 동일, 너무 짧으면 context switching overhead 증가(called processor sharing)
- SRTN
    - 남은 running time이 짧은 걸 기준
    - 장점: SPN과 비슷함
    - 단점: running time 예측으로 인한 overhead 발생, starvation(남은 running time 긴 건 실행 안됨)  발생
- HRRN
    - 1 + waiting time / burst time을 기준으로 함
    - 단점: 추정하는데 overhead 발생
- Priority
    - Process 생성 시 결정된 우선순위 기준으로
    - 단점: 우선순위 낮은 process의 starvation

### hybrid

- Multi level Queue
    - queue를 여러 개 사용하여 Process 생성 시 사용하는 queue가 정해짐
    - queue마다 preemptive, non-preemptive 설정 가능
- Multi level Feedback Queue
    - queue를 여러 개 사용하며 queue마다 priority가 존재함, queue마다 time slice 다르게 설정하여 starvation 줄임
    - 만약 process가 preempt 되면 running time이 긴 process이므로 priority가 낮은 queue로 이동됨
    - 만약 i/o를 하면 CPU 사용 시간이 적은 process로 간주되어 priority가 높은 queue로 이동됨
    - 장점: running time 예측, aging 계산 없이 비슷한 효과를 얻을 수 있음, burst time 줄어듦, adaptability 증가


- 이 글은 공부한 내용을 정리한 글으로 오류가 있을 수 있습니다. 오류를 발견하면 알려주세요!
