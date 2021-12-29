---
title: "System Engineering"
excerpt: "socio-technical system, system engineering definition and steps"
date: 2021-112-29
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
toc: true
toc_sticky: true
---

# 2. System Engineering

게시여부: No
날짜: 2021년 12월 19일
카테고리: SWEngineering

이 글은 소프트웨어공학개론 내용 정리 카테고리의 두 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요. 

## System 및 Sociotechnical system의 definition

### System

- 정의: purposeful collection of inter-related component for some common object  
즉 공통된 주제를 가지고 모여진 component의 집합이라는 뜻

### Sociotechnical system

- system의 구성요소에 법률, 규제 및 사람 또한 포함하는 것이 특징
- Sociotechnical stack  
society(법률, 규제) → organization(business rule, norm) → business(목표) → application(특정 function) → communication & data management(middle-ware, remote system에 존재하는 DB access 등) → OS → equipment(HW)  
SW Engineering: application ~ OS  
System Engineering: organization ~ equipment  
Socio-technical Engineering: society ~ equipment
    - SW가 organization에 미치는 영향
        - process change: business를 변경하고 이에 따라 관련 구성원을 훈련시켜야 한다. 이 때 구성원의 반발이 존재할 수 있다.
        - job change: 일을 하는 방식이 변경될 수 있고 해당 일을 처리하던 구성원을 de-skill할 수 있다. 즉 구성원은 전문 지식이 필요 없이 SW를 보조하는 형식으로 일하는 방식이 변경된다. 이 때문에 해당 분야의 전문가에 해당하는 구성원의 반발 또한 존재할 수 있다.
        - organizational change: SW를 관리하는 조직의 정치적인 힘이 강해진다.
- Sociotechnical system의 특징
    - Emergent property  
    전체로써의 특징으로 component의 relationship의 결과라고 할 수 있다. 개발 component에서는 나타나지 않지만 component를 모두 합쳤을 때 나타나는 특징이다.  
    Volume(total space of the system), reliability, security, repairability, usability 등이 예시가 될 수 있다.  
    Reliability에 관련된 내용으로는 fault propagation이 있다. HW propagation의 경우 bath tub 형태(x축: 시간, y축: fault ratio)로 나타난다. 시간이 지날수록 기하급수적으로 fault가 증가하는 이유는 HW의 lifetime이 한정되어 있고 부품의 lifetime이 끝남으로써 오류가 증가하기 때문이다. SW propagation은 update를 진행할 때마다 fault가 증가하고 감소하나 전체적인 fault는 시간이 지날수록 증가하는 형태를 가진다. 비록 HW처럼 lifetime이 존재하지는 않지만 update를 진행할수록 처음 의도했던 SW에서 벗어나고 component간 interaction 방식이 수정되면서 전체적인 fault는 증가하게 된다. 따라서 SW update를 진행할 때는 해당 change가 필요한지, 파급 효과는 어느 정도인지에 대한 충분한 검토가 필요하다.
    - Non-determinism  
    같은 input에 대해 다른 output이 발생하는 것을 뜻한다. 사람이 system에 포함되면 이러한 현상이 발생할 수 밖에 없다. 사람은 항상 똑같이 행동하지 않기 때문이다. 또한 요구에 따라 system이 변하기 때문에 output이 달라질 수 있다.
    - Success criteria  
    이해당사자마다 성공의 기준이 다르기 때문에 성공의 여부를 측정하기 힘들다.

## System Engineering Definition 및 각 단계에 대한 간략한 설명

Concept Design → Procurement → Development → Operation

### Concept Design

- 왜 이 system이 필요하고 목적이 무엇인지 정한다.(feasibility study, overall vision 제시)
- system vision & feature를 procurement에 전달, outline requirement를 development에 전달, user infomation을 operation에 전달한다
- requirement Engineering보다 앞선 단계로 일부 단계는 겹치기도 한다.(design과 requirement는 일부 겹친다.)
- SW 비전문가가 high level description을 제작하는 단계이다.
- 단계  
Concept formulation(system type 정의, needs 분석) → Problem understanding(stakeholder의 문제 이해) → System proposal development(가능한 system 제안) → Feasibility study → System structure development ⇒ System vision document

### Procurement

- Organization의 needs를 해결하기 위한 system을 습득한다.
- organization에 존재하는 다른  system, 외부 규제, external competition, business re-organization, budget을 고려하여 system을 결정한다.
- system의 scope, budget & timescale, high level requirement를 이용하여 type of system, potential supplier를 결정한다.
- equipment & SW updates를 development에 전달
- component 구매 여부에 따라 단계가 달라진다.
    - off-the-shelf(COTs)  
    concept design → access approved application → select system requirement → place order for system  
    특정 고객을 타겟으로 만드는 것이 아니라 동일한 형식으로 고객에게 전달되는 SW이다. COTs를 구매하는 것이 일반적으로 직접 develop하는 것보다 저렴한 편이다. 따라서 가능한 한 develop을 최후의 수단으로 삼는 것이 바람직하다.
    - configurable system  
    concept design → market survey(결과인 refine requirement를  choose supplier 단계에서 사용) → choose system shortlist → choose supplier → negotiate contract or modify requirement  
    시스템 사용자가 개발자의 도움을 받지 않고 personalize하기 쉽게 개발된 시스템을 의미한다. 따라서 개발하고자 하는 system에 맞는 configurable system을 찾고 customer와 협상을 통해 요구사항을 조절하는 과정이 필요하다.
    - custom system  
    concept design → define requirement → issue request for tender → choose supplier → negotiate contract or modify requirement  
    특정 고객을 대상으로 만들어지는 SW다.
- 어떤 supplier를 결정하느냐에 따라 프로젝트의 성공 여부가 달라질 수도 있기 때문에 신중하게 선택하는 것이 중요하다.
- specification of requirement는 계약에 포함되는 사항이고 이를 바탕으로 프로젝트의 성공 여부를 판단하는 경우가 많기 때문에 충분한 시간을 갖고 작성하는 것이 필요하다.

### Development

- 주로 parallel development에 도움이 되도록 plan driven  방식을 사용한다.
- 단계  
requirement engineering → architecture design → requirement participation → subsystem engineering → system integration → system testing → system deployment → decommission   
세부 사항은 차후 단원 별로 자세하게 다루게 된다.
- Design spiral  
domain & problem understanding → requirement elicitation & analysis → architecture design → requirement partition → review & assess 를 반복

### Operation

- 사용하기 편리해야 하고 flexible해야 한다.
- Human error
    - 사람이 system을 사용하면서 발생하는 error이다. 실수로 허용된 range 외의 값을 입력하는 등이 예시가 될 수 있다. human error에 대해서는 두 가지 관점이 존재한다.
    - personal approach  
    사람이 잘못한 것으로 판단하여 error를 유발시킬 시 주의를 주고 불이익을 주는 등 사람이 조작할 때 실수를 줄이도록 관리한다. System Engineering에서 추구하는 방식은 아니다.
    - system approach  
    사람이 실수할 수 있다는 것을 기본 전제로 하고 system의 차원에서 이 실수가 system을 파괴하기 전에 막을 수 있도록 다양한 안전 장치를 사용한다. 예를 들어 multi barrier 등의 모델을 사용할 수 있다. 사람이 잘못된 값을 입력하면 ‘정말로 입력하시겠습니까?’ 등의 주의 문구를 보여주는 식이다. 그러나 swiss cheese 모델에서 볼 수 있듯 이러한 barrier로도 막지 못하는(치즈의 구멍을 뚫고 지나가는) error는 분명 존재할 수 밖에 없다.
- Evolution
    - technique, business의 변화로 인해 change는 피할 수 없다.
    - change는 cost를 유발하는 행위이기 때문에 다방면에서 검토가 필요하다. 그러나 interaction로 인해 예상하지 못한 다른 곳에서 추가적인 문제 발생, 처음에 왜 그렇게 설계했는지 알 수 없어서 수정이 힘든 문제 등이 발생한다. 이를 예방하기 위해서는 dependability를 잘 정리해야 한다.
    - system의 lifetime은 여러가지 요소에 의해 영향을 받는다. 예를 들어 해당 시스템 전문가의 부재, 새로운 system으로 대체하는 것이 비용적인 측면에서 효율적임, 투자 비용 회수, 변화로 인한 risk, system dependency 등에 의해 lifetime이 결정될 수 있고 해당 요소들을 고려했을 때 새로운 system을 사용하는 것이 바람직하다고 판단되면 해당 system은 decommission 된다.

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
