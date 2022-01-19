---
title: "7 Architecture Design"
excerpt: "Architecture Design 정의, 장점, 선택 시 고려사항, Pattern"
date: 2022-01-17
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
---

안녕하세요.   
이 글은 소프트웨어공학개론 내용 정리 카테고리의 일곱 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

- Development 부분에서 requirement engineering 이후 진행되는 단계이다.
- model vs design  
model은 어떻게 구현하는지에 대해 다루기보다는 무엇을 구현하는지(What)에 집중한다. requirement engineering 중에 사용된다.  
반면 design은 어떻게 구현할지(How)에 대해 집중하고 requirement engineering 이후에 진행되는 단계이다.  
그러나 main component, relationship이 modeling할 때 결정되기 때문에 model과 design은 밀접한 관계가 있다.
- Architecture reuse  
domain이 동일한 architecture는 비슷한 형태를 가지기도 한다. 따라서 core architecture 주위에 해당 프로젝트에만 존재하는 특정한 application product line을 덧붙이는 형식으로 존재하는 core architecture를 재활용할 수 있다.

## Definition and Advantage of Architecture Design

= sub-system identification, control & communication related to framework가 정의되는 것

- output은 SW architecture description이다.
- 장점
    - stakeholder와 소통을 하는데 도움을 준다.
    - system을 분석하는데 사용된다. 특히 non-functional requirement를 만족하는지 분석할 수 있다.
    - large-scale of reuse가 가능하도록 한다. 통용되는 design pattern을 사용할 수 있기 때문이다.
- conflict
    - large-grain component: performance ↑ maintainability ↓  
    component의 크기가 커지면 component간 소통이 줄어들기 때문에 연산 속도가 빨라지는 경향이 있다. 그러나 관리의 측면에서는 하나의 component 내에 많은 기능이 포함되어 있으면 불리하다.
    - redundant data: availability ↑ security ↓  
    불필요한 데이터를 중복하여 저장한다면 특정 부분이 구동되지 않을 때 다른 부분을 구동하면 되므로 사용자가 원할 때 서비스를 제공할 수 있는 availability가 높아진다. 그러나 데이터를 여러 곳에 저장하므로 security는 떨어진다.
    - localize safety-related data: safety ↑ performance ↓  
    safety에 연관된 데이터를 특정한 component 에서만 다룸으로써 safety를 높일 수 있다. 그러나 이를 위해 해당 component에 접근해야만 하므로 performance는 낮아진다.
- Architecture Representation  
simple, informal diagram을 사용한다면 모호함과 소통의 문제가 발생할 수 있다. 따라서 표기 방식을 정의하기도 한다.
    - C2: 같은 레벨에 있는 component의 모임을 다루기 쉽다. 그러나 같은 레벨 내 relationship을 표현하기에는 어려움이 있다.
    - Wave: component간 relationship을 나타내는데 용이하나 표기가 복잡하다.

## Design Decision

system type마다 다르기는 하나 어떤 design을 사용할 지를 정할 때 아래와 같은 사항을 고려할 수 있다.  
1. generic application이 존재하는가?  
2. system이 HW 상에 어떻게 분포되는가?  
3. 어떤 architecture pattern, style을 사용하는가?  
4. component의 operation 제어를 위해 어떤 기법을 사용하는가?  
5. architecture가 어떤 형식으로 문서화되는가?  
6. architecture organization 중 non-functional requirement를 가장 만족하는 것은 어떤 것인가?  
7. structure component가 어떻게 sub-system으로 나뉘어져 있는가?  
8. structure를 짜기 위해 어떤 approach를 사용하는가?

## Architecture View

어떤 view, notation이 design과 document struct에 도움을 주는지 생각해봐야 한다. 하나의 architecture model은 하나의 view만 보여주는 것이 원칙이다.

- 4 + 1 view model이 많이 사용된다.  
4 + 1 view model을 사용하면 필요한 대부분의 view를 모두 나타낼 수 있다.
    - logical: key abstraction을 object나 object class로 나타낸다.
    - process: run time 동안 어떻게 interacting process가 구성되는지 나타낸다.
    - develop: 개발할 때 시스템을 어떻게 나눌지 나타낸다.
    - physical: HW와 SW component가 어떻게 분배되어 있는지 나타낸다.
    - use case(+1 담당)

## Architecture Pattern

representing, sharing and reusing

- pattern은 잘 완성된 design practice에 대한 설명이다.
- pattern은 언제 해당 패턴이 사용되어야 하고 어느 상황에서는 유용하지 않을지 명시해야 한다.
- 테이블, 다이어그램을 통해 나타낼 수 있다.
- 다양한 종류의 패턴을 알고 현재 개발하고 있는 시스템에 적절한 패턴을 사용하는 것이 개발자로서 지향해야 하는 바이다.

### Layered architecture

- sub-system의 interface에 집중하는 패턴이다. 한 layer가 변화하면 인접한 layer만 영향을 받는다. lowest level layer가 core service이다.
- 사용: different layer에 대한 incremental development 진행, 존재하는 시스템에 새로운 facility 추가, several team이 개발 진행, multi-level security 필요 시스템
- 장점: interface만 유지되면 layer 전체 변경 가능, dependability 위해 redundant facility를 추가 가능
- 단점: clean separation 힘듦, high layer가 바로 low layer랑 interact 하는 등 communication을 체계적으로 하기 힘듦, performance 떨어짐

### Model-View-Controller(MVC)

- 보이는 것(presentation)과 interaction을 분리한 패턴이다. view, interact 방식을 여러 개 사용할 수 있다. 3개의 phase로 나뉘어져 있다.  
Model: data, operation 관리  
View: 사용자에게 보이는 data presentation 관리  
Controller: user interaction 관리
- 사용: 한 가지 시스템에 대해 view, interact 방식이 복수로 존재, interaction & presentation이 requirement에서 정확히 정해지지 않은 시스템
- 장점: representation과 별개로 data 변경 가능, 다양한 presentation 사용 가능
- 단점: data, interaction이 단순한 시스템의 경우 MVC 구현 위해 코드가 더 늘어날 수 있음

### Repository architecture

- 모든 데이터가 중앙에서 관리되고 component간 직접 interact를 하지 않는다.
- 사용: 정보의 양이 많은 경우, 데이터를 오랫동안 저장하는 시스템, data-driven system
- 장점: component가 다른 component와 독립적(DB와만 소통하기 때문에 타 component의 존재 여부와 상관없음), 한 component가 변경한 사항이 모든 system에 영향을 줄 수 있음(DB에 저장된 데이터를 변경하면 해당 DB를 system에서 공통적으로 사용하기 때문), data consistency가 뛰어남(공용 DB를 백업하면 되므로)
- 단점: single point of failure, 비효율적, repository를 분산하기 힘듦

### Client Server architecture

- system의 function이 service에 organize되어 있다. server에서 client에게 function 제공 가능하다.
- 사용: 공유된 DB, system에 대한 부하가 변화함
- 장점: server가 분산됨, general function을 모든 client가 사용 가능
- 단점: service가 single point of failure, 네트워크에 따라 service performance 예측 불가

### Pipe & Filter architecture

- 파생된 패턴이 많은 패턴이다.(예시: transform이 순차적으로 일어나는 경우 batch system이 된다) interactive 상황에서는 사용하지 못한다.
- 사용: data processing
- 장점: transformation reuse 가능, 패턴이 이해하기 쉬움, 다양한 business의 struct workflow에 맞도록 진행됨, evolution이 간단함, sequence & concurrent develop 모두 가능
- 단점: communication format을 사전에 정의해야 함, parsing하는 overhead 존재

## Application Architecture

generic application architecture = 특정 requirement를 만족시키기 위한 configured & adapted architecture

- architecture design의 가장 첫 단계이다.
- design 중 checklist로 적용할 수 있다.
- team에서 할 일을 분배할 때 참고할 수 있다.
- reuse 정도를 파악하는 기준이 될 수 있다.
- 특정 application types에서 사용되는 단어를 이해할 수 있다.

### Data processing

- data-centered system에서 사용한다. (DB가 크다)
- data-flow diagram으로 나타낸다.  
input → process → output 형식

### Transaction processing

- user가 보기에 목표를 만족하기 위한 어떠한 coherent sequence of operation은 다 transaction asynchronous requirement이다.
- input/output processing ↔ application logic ↔ transaction manager ↔ DB
- Information system architecture
    - layered architecture(user interface, user communication, information retrieval, system DB) 사용 가능
    - web based architecture(interface가 web browser로 진행됨, server가 application server & logic, DB server는 information 담당)
- Resource allocation
    - e-commerce와 같은 시스템
    - layer마다 각자 다른 server를 둘 수 있음

### Language processing

- language → output 형태일 때, interpreter, algorithm & data 설명하는 방식이 가장 효율적일 때 사용
- repository architecture, pipe&filter compiler 패턴을 사용하여 구현 가능  

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 하신다면 댓글로 알려주시길 바랍니다.
