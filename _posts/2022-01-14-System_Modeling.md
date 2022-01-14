---
title: "6 System Modeling"
excerpt: ""
date: 2022-01-14
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
---

이 글은 소프트웨어공학개론 내용 정리 카테고리의 여섯 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

모델은 시스템을 추상화하여 간단하게 표현하고 특정 시각에서 시스템을 표현함으로써 이해당사자들의 시스템에 대한 이해도를 높이는데 사용됩니다. 또한 현존하는 시스템을 분석할 때 사용되기도 합니다.

Agile과 상반되는 방식으로 **model-driven engineering(MDE)** 방식이 존재합니다. 이 방식에서는 모델을 잘 정의해 놓는다면 미리 만들어져 있는 translator를 이용하여 구현을 자동을 진행합니다. 이에 대해서는 해당 글 하부에서 자세히 다루겠습니다.

## Definition of Model

= process of development abstract model of system with specific view

- 모든 모델은 **간단(simple)** 해야 하고 **특정 관점(view)** 에서 시스템을 바라봐야 한다.
- 시스템을 graphical notation(UML 등)으로 표현하는 방식이다.
- 분석가와 customer가 시스템을 이해하는데 도움을 준다.
- existing system modeling  
requirement engineering(이전 글에서 다룸) 단계에서 이미 존재하는 시스템을 모델링한다. 이를 통해 존재하는 시스템을 clarify하고 해당 시스템의 장단점을 논의하는 기초로 사용된다. 존재하는 시스템의 장단점을 바탕으로 새로운 시스템을 고안한다.
- new system modeling  
마찬가지로 requirement engineering 단계에서 진행되고 proposed requirement를 stakeholder에게 설명할 때 사용된다. engineer는 design 및 서류 작성 단계에서 참고한다.

## UML

- 13 종류의 diagram 이 있으며 이를 활용하여 모델링을 진행할 수 있다.
- 흡사 표준으로 사용되고 있기 때문에 개발자로써 UML의 종류를 익히는 것이 중요하다.
- UML은 **object oriented** 에 집중하고 있기 때문에 functional oriented에서 사용하는 data flow, data related process에 관련된 diagram은 존재하지 않는다. 대신 activity diagram을 사용할 수 있다.
- UML에는 context diagram이 존재하지 않기 때문에 deployment diagram을 대신하여 사용한다.

## System Perspective

모델마다 시스템을 바라보는 관점이 있다. 따라서 시스템을 관찰하는 관점마다 전형적으로 사용되는 모델을 함께 설명하고자 한다. 각 diagram에 대해 이미지를 함께 찾아보면 더욱 이해가 잘 될 것이다. 한 가지 모델을 여러가지 다이어그램으로 표현할 수 있고 시스템의 한 측면에 대해 모델링을 진행한 결과를 시각화하는 방식이 다이어그램이다.

### Interactive perspective

- Interaction model: user-system(user requirement에 대한 이해 높임), system-system(communication problem에 집중), component-component(requirement performance, dependability 파악)간 interaction에 집중한 모델이다.
- diagram
    - Use case: UML, requirement elicitation에 효과적, discrete task
    - Sequence: UML, actor와 object 간 interaction 표현에 효과적, 특정 use case에 대한 sequence of interact를 표현

### Structural perspective

- Structural model: 시스템의 organization을 component와 relationship으로 표현하는 모델이다. static model일 수도 있고 dynamic model일 수도 있다. system architecture를 구상할 때 작성한다.  
static model: structure of system design을 표현한다.  
dynamic model: organization of system when executed를 표현한다.
- generalization: related class(inheritance, subclass, super class)만 표현하는 모델이다. kind of, is a로 나타낼 수 있다.
- object class aggregation: class가 다른 class와 어떻게 관계를 맺고 있는지에 대해 표현하는 모델이다. part of로 나타낼 수 있다.
- diagram
    - class diagram: object-oriented system일 때 class와 association간의 관계를 표현한다. object는 실제 세상의 물체인 경우가 많다.

### Behavioral perspective

- Behavioral model: 실행할 때 어떻게 행동하는지(when execute), 어떤 결과가 나오는지(what happen)에 집중하는 모델이다.
- data vs event  
data: 해당 값이 계산에 사용되는가?  
event: 해당 값이 계산에 사용되지 않고 다른 state로 변화되도록 trigger하는가?
    - Data driven model  
    business system인 경우 data driven인 경우가 많다. sequence of action을 보여준다. end-to-end process를 보여주는 데 효과적이다.
        - diagram: activity model, sequence diagram
    - Event driven model  
    real time system인 경우가 많다. data process보다는 event에 대한 반응이 주를 이루는 시스템에 사용된다. state machine model& event 등이 예시이다.
        - diagram: finite state diagram

## Model Driven Engineering(MDE)

앞서 말했다시피 프로그램을 작성하는 것보다는 모델을 작성하는데 집중하는 방식이다. 

- 장점: 높은 수준의 추상화(abstraction) 가능, 자동 완성 코드, platform independent
- 단점: 시스템 자체를 만들기보다는 추상화를 위한 engineering, 자동 완성을 위한 translator 만드는 게 더 많은 시간과 노력이 듬
- Model Driven architecture가 MDE의 전제조건이다.
- transformations  
computation independent model(CIM) + domain specific guide line → translator1  
translator1 + 사람 → platform independent model(PIM)  
translator1 + PIM → translator2  
translator2 + platform specific patterns & rules → platform specific model(PSM)  
translator2 + PSM → translator3  
translator3 + language specific patterns → executable code
- 주로 CIM은 domain 전문가가, PSM은 platform 전문가가 제작하는 경우가 많기 때문에 개발자는 PIM을 잘 설계하면 전체 시스템을 완성할 수 있다는 장점이 있다.
- 하나의 PIM을 기반으로 하여 여러 PSM translator를 이용하여 다양한 플랫폼에서 동작하는 코드를 작성할 수 있다.
- Agile과의 관계
    - agile은 executable code를 만드는데 집중하는 반면 MDE는 model을 만드는데 집중하므로 상반되는 개념이라고 볼 수 있다.
    - 만약 model을 통해 완벽한 코드를 결과로 얻을 수 있다면 Agile 방식에서도 사용할 수 있을 것이다.
    - 이를 위해 executable UML을 사용할 수 있다.

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.

