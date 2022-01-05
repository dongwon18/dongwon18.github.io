---
title: "3. Software Process"
excerpt: "Fundamental SW activity, Process models & activities"
date: 2022-01-05
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
toc: true
toc_sticky: true
---

안녕하세요.
이 글은 소프트웨어공학개론 내용 정리 카테고리의 세 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

## Software Process

- Software Process는 System Engineering 내 Development의 한 단계이다.
- structured set of activity to develop SW system
- 다양한 모델이 존재하며 각 모델은 specification, design and implementation, validation, evolution 4 단계를 필수적으로 포함한다.

## Models

### Plan Driven

- 정해진 plan에 따라 개발을 진행한다. 초반에 사용자의 요구사항이 확립되는 경우에 주로 사용한다.
- 대표적인 예로는 waterfall model이 있다.
- 장점
    - interface를 초반에 정의할 수 있다.  
    plan이 이미 결정되어 있기 때문에 interface 또한 결정하기 쉽다.
    - visibility가 향상된다.  
    각 단계에 뚜렷한 구분이 존재하므로 프로젝트 전체에 대한 visibility가 높다.
    - 문서화가 쉽다.  
    각 단계가 뚜렷하게 구분되기 때문이다.
    - manage가 쉽다.  
    visibility가 높고 문서화가 잘 되어 있기 때문에 비교적 프로그램을 관리하는 것도 용이하다.

### Waterfall

- 정해진 단계를 순서대로 진행하게 된다. 앞서 다룬 plan driven model의 장점을 가진다. 그러나 순서대로 진행되기 때문에 이전 단계로 돌아가기 힘들다는 특징이 있다.
- Requirement definition → System & SW design → Implementation & Unit-testing → Integration & System testing → Operation & Maintain

### Incremental

- specification → development → validation을 여러 번 반복하는 model이다.
- plan driven과 반대되는 개념의 model이라 할 수 있다.
- user의 요구사항이 계속 변경되는 경우에 유용하게 사용될 수 있다. iteration을 반복하기 때문에 이전 iteration에서 부족했던 부분을 이후 iteration에서 보강하는 등의 대처가 가능하다.
- 초기 iteration의 결과는 prototype으로 사용될 가능성이 높다.
- 그러나 여러 번 피드백과 수정이 반복되기 때문에 프로젝트 전체의 visibility와 문서화가 어렵다는 특징이 있다. 이러한 특징으로 인해서 managing team에서 시스템을 이해하기가 어렵고 따라서 manage도 하기 어렵다. 또한 contract를 작성하기 모호하다는 단점도 존재한다.
- 장점
    - 변화를 수용하는 비용이 비교적 적다.
    - 빠른 delivery가 가능하다.  
    필요에 따라 version을 나눠서 배포할 수 있기 때문이다.
    - project 전체에 대한 실패 확률이 낮다.
    - 우선 순위가 높은 부분부터 개발하여 test를 진행하기 때문에 중요한 부분은 여러 version을 거치면서 계속 test되기 때문에 reliability가 높아진다.
- Outline description → (specification → development → validation) → 1st version, ..., last version

### Reuse-oriented

- 직접 개발하는 것보다 존재하는 component를 사용하는 것이 비용적, 시간적, 질적이 측면에서 효율적인 경우가 빈번하다. 이렇게 존재하는 component를 사용하는 model이 reuse-oriented model이다.
- component의 종류: web service, collection of objects, CoTs, open-source
- user의 요구사항와  100% 일치하는 component가 존재하는 경우는 거의 없기 때문에 customer와의 requirement 조절이 필요하다.
- component를 직접 제작하지 않았기 때문에 전체 system에 대한 control이 힘들 수 있다.
- Requirement Specification → SW discovery / SW evaluation → requirement Refine → configure application system / adapt component / develop new SW → integrate SW

## Process activities

- Specification, design, implementation, evaluation을 끼워 넣는 형식(inter-leaved, incremental) 혹은 연속적(sequence)으로 진행된다.

### Specification

- 진짜로 이 서비스가 필요한지(feasibility study), 제약 조건은 무엇인지(constraints) , 높은 수준의 요구 사항을 획득(requirement engineering)하는 단계이다.
- Requirement Elicitation & Analysis ↔ requirement specification ↔ requirement validation
- 각 단계의 결과물로 system description, user & system requirements, requirement document 가 도출된다.

### Design & Implementation

- system specification의 내용을 executable system으로 변환하는 과정이다.
- Design
    - architecture design을 진행한다.
    - input: platform information, requirement specification, data description
    - design activity: architecture design → interface design → component design  
    architecture design, component design 결과를 토대로 DB design 진행
    - output: system architecture, DB specification, interface specification, component specification
- Implementation
    - design한 결과물을 바탕으로 실제 코드를 작성하는 과정으로 debugging까지 포함된다.
- Validation
    - validation & verification을 모두 진행하는 과정이다.
        - validation: customer의 요구사항과 시스템이 일치하는지 확인하는 과정이다. testing이 해당한다.
        - verification: document와 시스템이 일치하는지 확인하는 과정이다. review, inspect가 해당한다.
    - testing sequence  
    component testing → system test, emergency test → acceptance test 를 반복
        - V model  
        테스트에 필요한 정보가 생성되는 시점에 testing plan을 작성하는 모델  
        requirement specification + system specification(acceptance test plan) → system specification + system design(system integration test plan) → system design + detailed design(sub-system test plan) → module & integrate & test → sub-system testing→ system testing → acceptance testing → service

### Evolution

- SW는 외부적 요인(business, technique, platform 변화로 인해)으로 인해 변화될 가능성이 항상 존재한다.
- define system requirement → access existing system → propose system change → modify system → new system
- modify system단계 후 define system requirement로 돌아가 반복된다.
- 변화에 대한 2가지 관점
    - change avoidance  
    prototype 등을 사용해서 불명확함을 줄여 변화 가능성 자체를 줄이자는 관점
    - change tolerance  
    change가 많을 것으로 예상되는 system에는 incremental development 사용하여 변화 수용에 대한 비용을 줄이자는 관점
- Prototype
    - development & deliver에서 초기 버전이 주로 prototype인 경우가 많다.
    - 간략한 시스템을 customer에게 빠르게 전달해 피드백을 받을 수 있다.
    - 불명확성 감소 → maintainability 향상
    - customer의 피드백→ usability 향상, closer match to user need, design quality 향상, developing effort 줄어듦
    - 모호한 점이 존재하는 부분에 대해 prototype을 제작하는 것이 효율적이다.
    - prototype은 non-functional 요소 고려 x, document 부족, 빠른 개발로 인한 structure 부실, standard 만족 x 등의 이유로 인해 prototype은 모호한 점을 해결하고 버리는 것이 적절하다.
    - exploratory prototype: prototype에 살을 붙여 실제 system을 만드는 방식으로 위의 이유로 인해 일반적으로 권장되지는 않는 방식이다.
    - establish prototype objective(prototype plan) → define prototype function(outline definition) → development prototype(executable prototype) → evaluate prototype(evaluation report)

### Process Example

- Boehm’s spiral model
    - risk based backtracking방식으로 object setting → risk assessment & reduction → engineering & development validation → planning 을 나선형으로 반복한다.
- Rational Unified Process
    - incremental development, reuse component 많이 사용, UML 등 이용 시각화가 특징이다.
    - 3가지 관점: dynamic, static, practice 존재
    - dynamic: inception(개념화) → elaboration(초기 아키텍처 구축) → contruction(시스템 완성) → transition(시스템 이전)
    - static: process workflow(business model, requirement analysis & design, implementation, test, deploy), supporting workflow(configuration management, environment)
    - practice: develop SW iteratively, manage requirement, use component-based architecture, visually model SW, verify SW quality, control change to SW
- Cleanroom SW development
    - defect avoidance를 주목적으로 하는 모델으로 increment development, formal specification(수학을 이용하여 표현), structured program(go to 문 사용 x), statistic verification(실제로 실행하지 않고 눈으로 검열), statistical testing이 특징이다.
    - formal specification, statistic verification 등으로 인해 developing team의 역량이 뛰어나야 사용할 수 있다.
    - formally specified system / develop operational profile → define SW increments → construct structured program → formally code verify → integrate increment  
    define SW increment → design statistical test → test integrate system  
      
이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
