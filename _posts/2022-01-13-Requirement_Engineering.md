---
title: "5 Requirement Engineering"
excerpt: "user/system requirement, func/non-func/domain requirement, 요구명세서, 명세화 활동"
date: 2022-01-13
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
toc: true
toc_sticky: true
---

이 글은 소프트웨어공학개론 내용 정리 카테고리의 다섯 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

System Engineering 내에서 concept design 후에 진행하거나 design과 겹치는 형식으로 진행되는 부분입니다. Develop의 첫 단계라고 볼 수 있습니다.

- Requirement = constraint & service에 대한 설명, object to be tested
- Requirement vs Goal  
Requirement는 검증의 대상인 반면 goal은 검증을 하지 않아도 되는 사항입니다.
- Agile에서는 기본적으로는 requirement document를 작성하지 않습니다. 이 때문에 전제 조건이나 제약 사항이 많은 프로젝트에서 Agile을 사용하기에는 적절하지 않을 수 있습니다. 그러나 간소한 버전으로 작성하는 방법도 존재합니다. 예를 들어 XP의 경우 scenario로 작성합니다.

## User & System Requirement

기술적으로 자세한 정도 등에 따라 요구사항의 종류가 나뉜다.

- User Requirement  
자연어, diagram 등을 활용하여 사용자가 이해하는 것을 목적으로 작성되는 요구사항이다. user requirement를 바탕으로 system requirement가 도출된다.  
사용자: client, client manager, end user, client engineer, contractor manager, system architect
- System Requirement  
구조화된 문서, contract에 사용되는 문서로 개발자가 개발에 필요한 내용을 자세히 기술한 요구 사항이다.  
사용자: system end user, client engineer, system architect, developer

## 3가지 Requirement

- functional
    - How?에 집중하는 요구사항이다.
    - user, system requirement는 functional requirement에 포함된다.
    - 지양하는 바를 명시한다.(적어도 이건 하면 안 된다를 정함)
    - type of SW, expected user를 바탕으로 정해진다.
- non-functional
    - constraint, system properties를 정의한다.
    - IDE, 개발에 사용하는 언어 정의 또한 해당한다.
    - functional requirement보다 더 critical한 요구사항으로 만약 지켜지지 않는다면 아예 시스템을 사용하지 못할 수도 있다. (반면 functional은 사용자가 요구하는 기능을 다르게 구현하는 정도이기 때문에 협의를 통해 조절할 수도 있다.)
    - 예시 및 측정
        - speed: response time, processed transaction
        - size: Mbytes, ROM Chip 개수
        - ease to use: 사용자 training time, help frame 개수
        - reliability: mean time failure, probability of unavailability
        - robustness: time to restart after failure, failure 유발 event 비율
        - portability: target dependent statement의 비율, target system 개수
    - 적용 예시  
    product: speed, reliability  
    organization: 지켜야 할 policy, procedure  
    external: 법률 등
- Domain Requirement
    - 시스템 자체에 대한 제약 사항 등으로 domain requirement를 만족을 하지 않는다면 해당 시스템을 특정 분야에서 아예 사용하지 못할 수도 있다.
    - domain requirement에서 functional, non functional requirement가 도출된다.
    - 도메인 담당자가 전문 용어로 소통하기 때문이 이 자체를 이해하는데 어려움이 있다.

## Requirement Documentation & Specification

### SW Requirement Document

- = official statement of what is required of the system development
- user, system requirement의 정의 또한 포함해야 한다.
- how에 대해 다루는 design과 달리 what을 다뤄야 한다.
- document의 user
    - customer: requirement specification을 확인하고 필요에 따라 요구사항을 변경하기 위해 사용
    - manager: 프로젝트와 developing process를 계획하기 위해 사용
    - system engineer: 어떤 system을 개발해야 하는지 알기 위해 사용
    - system maintenance engineer: 시스템과 타 시스템, 시스템 내 component간 relationship을 이해하기 위해 사용
    - system test engineer: validation test를 만들기 위해 사용
- Document 포함 내용
    - 기업, 조직마다 다르다.
    - incremental develop에서는 document의 detail이 plan-driven보다는 떨어진다.
    - 이 글에서는 IEEE standard에서 정의한 내용을 소개한다.
        - Preface  
        document간 relationship과 document의 version을 설명한다.
        - Rationale & Summary of version
        - Introduction  
        왜 이 시스템이 필요한지, 기능은 무엇인지, 어떻게 작동하는지 간단하게 기술한다. organization의 overall business와 commissioning 방법 또한 포함한다.
        - Glossary  
        해당 document에 사용된 technical term을 정의한다.
        - User requirement definition  
        product와 process에 적용되는 standard에 대한 설명과 non-functional requirement 또한 설명한다. 규칙이 있는 notation, 자연어와 diagram으로 기술되어 있다.
        - System architecture  
        system에 대한 high level overview로써 기능의 distribution을 설명한다.
        - System requirement specification  
        다른 시스템과의 interface에 대해 다루고 functional & non-functional requirement에 대해 자세하고 기술적인 관점에서 서술한다.
        - System model  
        graphical system, system component, environment의 relation을 보여주는 파트이다. 다양한 모델이 존재하고 예시는 object model, data-flow model 등이 있다. 모델에 대해서는 이후 글에서 자세히 다룰 예정이다.
        - System evolution  
        기본적인 가정, 하드웨어, user needs 변화에 따라 시스템이 어느 정도 수정될 지에 대해 예측한 바를 기술한다. design시 기피해야 할 사항 또한 정의한다.
        - Appendices  
        HW(최소 사양, 최적 사양), DB에 대한 설명을 포함한다.
        - Index  
        문서에 대한 인덱스를 작성한다. 이 때 그림과 표에 대한 인덱스 또한 작성한다.

### Requirement Specification(명세화 활동)

= user & system requirement를 requirement document에 작성하는 활동

- 작성 방법
    - natural language  
    자연어로 작성하면 이해하기는 쉽지만 모호한 점이 많다. 따라서 다양한 방식을 사용하여 명세화를 진행한다.  
    대체로 diagram, tabular specification과 혼용되어 사용된다.  
    모호함을 줄이기 위해 format을 사용하는 것이 권장된다.  
    예) shall for mandatory, should for desirable, text highlighting, 컴퓨터 전문 용어 자제, 왜 해당 요구사항이 필요한지 등 기술
    - formed based specification  
    function, entity 정의, inputs, outputs 정의, 계산에 필요한 정보에 대한 설명, action, pre&post condition, side effect를 포함하여 자연어로 기술하는 방식이다.
    - structured natural language  
    standard form template에 따라서 자연어로 작성하여 모호한 점을 줄인다.
    - design description  
    프로그래밍 언어 등을 사용하여 작성하는 방법이나 customer가 이해하기 어렵기 때문에 사용 빈도가 낮다. interface specification에 효과적이다.
    - graphical notation  
    functional requirement 정의를 위해 사용된다. UML이 통용되는 대표적인 방식이다. UML에는 총 13가지 diagram이 존재한다. 자연어의 모호함을 줄이기 위해 자연어로 작성된 명세서를 보충하는 식으로 많이 이용 된다.
    - mathematical specification  
    finite-state machine(FSM) 등의 수학적 개념을 활용하여 표현한다. 애매모호한 사항을 줄일 수 있다는 장점이 있지만 이해가 어렵다는 단점도 존재한다.

## Requirement Engineering Process

정확한 단계와 명칭은 application domain, organization에 따라 다를 수 있다. 그러나 기본적인 단계는 requirement elicitation, analysis, validation, management는 공통적으로 포함된다.

- 해당 process를 spiral view로 이해하는 방식(elicitation → specification → validation 반복)  
business requirement specification → feasibility study → user specification elicitation → user requirement specification → prototype → system requirement elicitation → system requirement specification & modeling → review

### Requirement Elicitation & Analysis

- technical staff와 함께 application domain 찾기
- system이 제공할 service, operational constraints 찾기
- end user, manager, engineer, domain expert, trade union 등이 해당 단계에 참여한다.
- discovery → classification & organizing → prioritization & negotiation  
discovery: 이해당사자와 소통하며 요구사항을 찾고 domain requirement도 찾는다.  
classification & organizing: 관련있는 요구사항끼리 grouping한다.  
prioritization & negotiation: requirement를 분석한 결과 conflict를 해결하고 이해당사자들끼리의 협의를 통해 요구사항 간 우선순위를 결정한다.
- 그러나 고객이 자신이 원하는 것이 정확히 무엇인지 모르거나 분석 중 요구 사항 변경, 새로운 이해당사자 등장 등의 문제가 발생한다.
- 요구사항을 얻어내기 위해 사용하는 방식
    - interview  
    가장 많이 사용되는 방식으로 선입견, 가정을 최소화하고 진행하는 것이 중요하다. 미리 질문을 정하는 closed interview 형, 자유롭게 질문하는 opened interview 형이 존재하고 주로 이 둘을 혼용하여 사용한다.
    장점: 전체적으로 customer가 어떤 일을 하는지, 어떻게 소통하는지 파악하기 좋다.  
    한계점: domain requirement를 얻거나 domain에 대한 이해도를 높일 수는 없다.(인터뷰 대상자가 전문 용어를 사용하여 이야기 하기 때문)
    - ethnography(참여적 관찰법)  
    일하는 모습을 관찰하면서 요구사항을 얻는 방식이다.   
    focused ethnography = ethnography + prototype
    장점: 실제로 customer가 얘기한 것보다 더 과정이 복잡하고 다양하다는 것을 직접 알 수 있다. 현 상황을 이해하는데 도움이 된다.   
    단점: 새로운 요구사항을 찾는데에는 도움이 되지 않고 그러한 용도로 사용하는 것 또한 부적절하다. 더 이상 관련 없는 예전 정보를 모으게 될 수도 있다.
    - scenario & stories  
    어떻게 시스템이 쓰이고 있는지 real-life example을 얻을 수 있다.  
    starting situation, normal flow of event, what can go wrong, concurrent activity, finishing situation 등을 포함하여 작성한다.
    - use case  
    scenario를 기반으로 하여 UML의 일종인 use case diagram을 그리는 방식이다.  
    interaction & actor를 정하여 diagram을 그리고 이를 customer에게 보여줌으로써 해당 역할에 관련 있는 활동을 가시적으로 표현할 수 있다.  
    가능성이 있는 모든 interaction을 다 표현해야 한다.  
    tabular description, sequence diagram으로 보충 설명을 추가할 수 있다.

### Requirement Validation

이것이 정말 user가 원하는 요구사항인가를 체크하는 단계이다.

- 확인 사항
    - validity: 요구사항을 충족하기 위한 best function인가?
    - completeness: 모든 요구사항이 포함되었나?
    - realism: 기술적으로 구현이 가능한가? 주어진 예산으로 가능한가?
    - consistency: 요구사항 간 conflict는 없나?
    - verificability: 요구사항을 만족했는지 평가할 방법이 있는가?
- 기술적 방법
    - (manual) review: client & contractor를 포함하여 진행한다. verificability(진짜로 테스트 가능한 사항인가?), comprehensibility(작성된 내용이 이해가 잘 되는가?), traceability(누가 해당 사항을 요구했는지 명시되어 있는가?), adaptability를 체크한다. requirement deifinition이 만들어지는 중에 진행된다.
    - prototyping
    - test-case generation  
    testability 확인을 위해 test case를 제작한다.

### Requirement Management

무분별한 요구사항 변화는 막대한 비용을 초래하고 프로젝트 진행에도 어려움을 야기한다. 따라서 Requirement engineering, system development 동안 요구 사항의 변경을 관리해야 한다.

- 각 요구사항을 추적하고 서로 상관관계를 유지하도록 한다.
- 변화에 따른 dependency 변화 추이를 분석한다.
- 요구 사항 변화를 위한 정해진 process가 있어야 한다.
- requirement evolution 과정  
initial understanding of problem → initial requirement → changed understanding of problem → changed requirement
- requirement management plan  
requirement identification → change manage process → traceability policies → tool support  
requirement identification: 다른 requirement와 뚜렷하게 구분이 되어야 한다(cross-reference 위해)  
change manage process: assess impact & cost of change  
traceability policies: requirement간의 relationship, requirement와 system 간 relationship을 분석한다.  
tool support: 다양한 툴이 변화를 지원하는지 확인한다.
- identify problem → problem analysis & change specification → change analysis & costing → change implementation → revised requirement  

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
