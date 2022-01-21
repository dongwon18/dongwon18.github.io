---
title: "10 Project Management"
excerpt: "planning, type of planning, process, project scheduling"
date: 2022-01-21
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
---

안녕하세요.   
이 글은 소프트웨어공학개론 내용 정리 카테고리의 열 번째이자 마지막 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

- project를 관리한다는 것은 project가 **계획에 맞춰 진행되고 있는지(on time, on schedule), 요구 사항대로 진행되고 있는지** 확인하는 것입니다.
- 이 때 budget, schedule이 제약이 있으므로 주어진 인적, 물적 자원을 효율적으로 활용하는 것이 중요합니다.

## SW Project Management Definition

- SW는 다른 분야와 다른 특성이 있어서 관리에 있어서도 차별을 둬야 한다. 이와 같은 사항 때문에 프로젝트를 관리하는데 어려움이 존재한다.
    - intangible: SW는 결과물을 눈으로 볼 수도 만질 수도 없다.
    - flexible: 건축물 등 다른 결과물에 비해 수정이 용이하다.
    - sane status: 다른 공학 분야와 달리 소프트웨어 공학은 일반적인 공학에 적용되는 규칙이 없다.
    - no process standard: 정해진 process가 없다.
    - one-off project: 한 번 만든 시스템은 한 번만 사용할 수 있다. 존재하는 시스템을 온전히 그대로 사용할 수 있는 경우는 거의 없기 때문이다.
- management에는 아래와 같은 행위가 포함된다.  
proposal writing, project planning & scheduling, costing, monitoring & review, personal selection & evaluation(팀원 선택 및 평가), report writing & presentation  
이와 같은 사항은 타 공학에도 적용된다.

## Project Planning

가장 시간이 많이 소비되는 단계로 프로젝트 초기 단계부터 서비스 배포할 때까지 전반적인 과정 동안 진행된다.

- resource available, work breakdown, schedule을 다룬다.

### Types of plan

- Quality plan: quality procedure, standard를 어떻게 지킬 지에 대한 계획 수립
- Validation plan: approach, resource, schedule for validation 수립
- Configuration management: configuration management procedure & structure 수립
- Maintenance: requirement change prediction(effect, cost 고려)
- Staff Development: how skill, experience of team developing

### Process

constraint 찾기 → project parameter assess → project milestones&deliverable 정의 → schedule draw → action → project progress review → parameter revise → schedule update → re-negotiate → 문제가 발생했다면 technical review, possible revision initiation 진행 및 schedule draw로 돌아가기

- milestone: 프로젝트 팀 내에서 다루는 한 단계의 결과물
- deliverable: customer에게 전달되는 결과물

### Project plan structure

- introduction: 프로젝트의 목표와 제약 등 명시
- project organization: 팀원 및 역할
- risk analysis: 일어날 수 있는 위험요소(possible risk), 가능성(likelihood), 위험 정도 줄일 수 있는 방법(reduction strategy)  
만약 일어날 가능성이 높은데 위험 정도를 줄일 방법이 없다면 프로젝트 전체의 방향 전환이 필요함
- HW&SW resource requirement: 개발을 위한 HW&SW, 만약 구매해온다면 그 가격과 delivery time
- work breakdown: 세부 단계로 나누고 milestone, deliverable을 정함
- project schedule: 단계 별 dependency, 기간 예측, 팀원 분배
- monitoring & reporting mechanism: management report 생성

## Project Scheduling

- project를 task 단위로 나눈다.
- 각 task의 소요 시간과 필요 자원을 예측한다. 하나의 task는 1 ~ 2주가 적당하다.
- time to market을 줄이기 위해 가능한 한 동시에 진행한다.
- **task별 dependency는 최소화**한다. 의존성이 낮다면 동시 진행이 가능하다.
- identify activity → identify activity dependency → [estimate resource → allocate people → create project chart → activity chart & bar chart] 반복
- 일반적으로 투입된 인원과 생산성은 비례하지 않는다. (Brook’s law)

### Bar chart & Activity Network

둘 다 plan을 가시화하기 위해 사용된다.

- Bar chart: schedule을 날짜와 함께 표기한다. delay 가능 시간도 표현한다.
- Activity network: task dependency, critical path를 나타낸다.

## Risk Management

- identify risk, plan to minimize effect를 진행한다.
- project(갑작스런 팀원 변경 등), product(quality, performance), business risk(procure organizing)를 전반적으로 파악한다.
- risk identification → risk analysis → risk plan → risk monitoring
    - risk identify → potential risk list 작성 → technique, people, organization, requirement, tool, under estimate 등 에서 찾음
    - risk analysis → prioritized risk list → possibility & seriousness 파악
    - risk plan → avoidance & minimization & contingency plan 수립
    - risk monitoring → risk assessment → access regularly 통해 effect change 파악 및 경과 manage progress meeting에서 다루기

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
