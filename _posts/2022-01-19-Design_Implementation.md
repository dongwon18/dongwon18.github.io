---
title: "8 Design & Implementation"
excerpt: "Object Oriented Design, Process of Design, Reuse, Open Source"
date: 2022-01-19
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
---

안녕하세요.  
이 글은 소프트웨어공학개론 내용 정리 카테고리의 여덟 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

- design & implementation의 결과로 executable SW가 도출됩니다.
- design은 SW의 component와 관계를 결정하는 creative activity이고 implementation은 이를 realizing하는 단계입니다.

## Object Oriented Design

- SW가 지향하는 3가지: **cost, quality, time to market**
- cost: 비용이 많이 들지만 maintain하는 데 도움이 되기 때문에 lifetime이 긴 시스템에 적용 시 비용적인 측면에서 이득을 볼 수 있다.
- quality: **cohesion**(응집력, 여러 component가 서로 얼마나 연관이 있는가?)은 **높고** **coupling**(결합력, 한 component가 바뀌었을 때 다른 component에 얼마나 영향을 주는가?)은 **낮은** 게 품질이 높은 SW라고 할 수 있다. OOAD(Object Oriented Analysis and Design)을 사용하면 앞서 얘기한 특성을 갖기 때문에 quality가 어느 정도 보장이 된다.
- time to market: reuse의 최소 단위가 object이다. 따라서 OOAD를 사용하면 reuse의 비율을 높일 수 있기 때문에 개발 시간이 적게 걸린다.
- 일반적으로 많은 노력이 필요하기 때문에 규모가 작은 시스템보다는 큰 시스템, lifetime이 긴 시스템에 사용한다.

### Object Modeling Technique

1. object modeling  
problem description, interview 진행, 관련 system 분석 → object, attribute, relationship 정의
2. dynamic modeling  
problem description(1단계와 겹쳐 생략하는 경우도 많음) → Scenario 작성 → Sequence chart & Event trace diagram 그리기 → 1단계에 정의한 각 object에 대해 STD(State transition diagram)작성  
class diagram을 정확히 찾아내는 게 목표이다.
3. functional modeling  
2단계에 정의된 각 activity에 대해 DFD(Data Flow Diagram) 작성  
원래 OMT는 DFD를 사용한다. 그러나 UML에는 존재하지 않는 diagram이므로 active diagram을 대신 사용한다.

## Process stage of Design

기관마다 다를 수 있지만 객체 지향 디자인을 사용할 때 대체적으로 다음과 같은 단계를 따른다. 해당 단계에서 사용하는 UML diagram과 엮어 설명할 예정이다.

### System context & Interaction

- 해당 시스템과 외부와의 관계를 정의한다.  
어떻게 필요한 정보를 주고 받으며 소통하는지 정하다.
- boundary를 정한다.  
시스템이 관장하는 부분이 어느 정도까지인지 명시한다.
- context model  
structured model, environment를 설명한다.
- interaction model  
dynamic model, interaction을 설명한다.
- diagram: context diagram, use case diagram

### Architecture

- major component와 interaction 방식을 정한다.  
architecture pattern을 이용하여 component를 구조화한다.
- diagram: package diagram

### Object class identification

이 단계부터 본격적으로 객체 지향 방식을 도입한다.

- class를 정의한다.  
해당 행위는 iterative하게 진행된다.  
class를 정하는 정해진 방법이 존재하는 것은 아니나 아래 사항을 고려하여 정하는 것이 일반적이다.
    - tangible한 물체 구별
    - role, event, interaction, location, unit 구별
    - grammatical analysis: 명사, 동사 구별
    - behavior: 무슨 행동인지 근거하여 연관된 object 연결
    - scenario를 기반으로 만듦

### Design model

- object, class와 이들의 relationship을 보여준다.
- static model: static structure
- dynamic: interaction
- 예시  
sub-system model: logically related group에 design이 organize 되는 방식을 얘기한다 (package model, logical model)    
sequence:  interaction의 sequence를 나타낸다 (sequence diagram)  
state: requirement에 어떻게 반응하는지, state transition을 나타낸다. transition 선마다 test를 만들어야 한다.  
그 외 use-case, aggregation, generalization 등

### Interface specification(접속 명세)

- interface를 정하고 나면 parallel development가 가능하기 때문에 interface를 정의하는 것이 필요하다.
- attribute가 아직 정해지지 않았기 때문에 상세하진 않으나 operation을 정하는 정도로 작성한다.
- diagram: class diagram

## Design Pattern

- 잘 정의된 model을 재사용하도록 주어지는 pattern이다.
- pattern은 충분히 abstract해서 필요에 따라 다양한 시스템에 적용할 수 있도록 작성하는 것이 좋다.
- 주로 이름, 문제점, 설명, 해결방식, 해결방식에 대한 설명, 결론 을 포함하여 작성된다.

---

## Implementation

- reuse, configuration management, host-target development가 구현 중 필요하다.

### Reuse

- abstraction level, object, component, system을 재사용할 수 있다. reuse level에 따라 선택한다.
- reusing cost: looking, assessing, buying, adapting, configuring, integrating 과정에서 발생하는 비용을 포함한다.

### Configuration management

- change managing process, integration시 도움을 받기 위해 사용한다.
- version management, system integration, problem tracking(bug report)
- 더 자세한 내용은 Project Management 글에서 다룰 예정

## Open Source Development

- 코드 작성자가 코드에 대한 소유권을 소유하는 것이 아니라 공동자산으로 생각하는 Free Software Foundation을 기반으로 한다.
- 사용자가 코드를 테스트할 뿐 아니라 수정까지 할 수 있도록 한다.
- SW 자체를 판매하여 수익을 올리기 보다는 공개한 SW를 유지보수 해주는 서비스를 통해 수익을 올리는 방향으로 business model이 변경되고 있다.
- 장점
    - 코드를 다양한 환경에서 테스트한 결과를 얻을 수 있다.
    - 개발자로서 오픈 소스에 기여했다는 자부심을 느낄 수 있다.
    - 필요한 부분을 reuse하면서 time to market, cost, quality의 측면에서 이득을 볼 수 있다.
- license model
    - GNU GPL: 오픈 소스를 사용해서 개발했다면 개발한 SW도 공개해야 한다. 가장 엄격한 모델이다
    - GNU LGPL: 오픈 소스를 사용해서 개발한 SW를 공개할 의무가 없다.
    - BSD: 어떠한 유지보수 결과도 공개할 필요가 없다. 판매도 가능하다.
- 예: Linux, MySQL, Android, Java, Apache web server 등

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
