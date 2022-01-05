---
title: "1 SW Engineering Introduction"
excerpt: "defi & cost of SW, good SW, application type, ethics"
date: 2021-12-19
categories: 
  - SWEngineering
tags:
  - Theory
  - SWEngineering
toc: true
toc_sticky: true
---

이 글은 소프트웨어공학개론 내용 정리 카테고리의 첫 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요. 

## SW Engineering의 definition

- 더욱 많은 시스템이 SW로 통제되고 있고 날이 갈수록 SW에 대한 관심도 및 중요도가 상승하고 있다. reliable, trustworthy, economically, quickly development를 위해 더욱 중요해지고 있다.
- SW Engineering은 SW에 관련된 theory, method, tool을 모두 다루는 공학이다. project managing, developing tool 또한 SW Engineering의 고려 대상이다.
- SW는 program 뿐 아니라 program에 연관된 document  또한 포함한다.

### Generic product vs Customized product

- generic product: stand alone인 경우가 많고 specification을 developer가 결정한다.
- customized product: 외부 프로그램과 소통해야 하는 경우가 많으며 specification을 customer 혹은 user가 결정한다.

## cost in SW Engineering

- SW cost > computer system cost인 경우도 빈번하게 발생한다.
- SW의 lifetime이 길어질수록 maintain cost가 증가한다.
- 따라서 SW Engineering에서는 cost-effective development를 추구한다.
- 직접 develop하는 것은 SW를 구매하는 것보다 훨씬 많은 비용을 발생시키므로 가능한 한 최후의 수단으로 developing을 하는 것이 좋다.

### model에 따른 cost 분포

- waterfall  
Specification(20%) → Design(20%) → Development(20%) → Integration and testing(40%)  
 전통적인 모델로 사용자의 요구가 초반에 명확하게 정해지는 경우를 가정한다.
- Iterative  
Specification(10%) → Iterative development(60%) → System Testing(30%)  
실제로는 사용자의 요구가 명확하게 정해지기보다는 바뀌는 경우가 빈번하다. 따라서 여러 iteration을 반복하며 점진적으로 개발을 진행하는 방식에 적합한 모델이다. 한 iteration내에서 specification → implementation → validation을 진행한다.
- component-based  
Specification(20%) → Development(30%) → Integration and Testing(50%)  
SW reuse의 빈도가 늘어나는 만큼 SW를 component 단위로 나눠 필요에 따라 reuse하거나 COTS하는 등 블록의 형태로 SW를 개발하는 모델이다. 그렇다 보니 필요한 부분은 직접 개발을 하지만 component를 합치고 제대로 동작하는지 테스트하는 비용이 많은 부분을 차지한다.
- long-lifetime  
Development(10%) →  System evolution(400%)  
Lifetime이 긴 SW의 경우 개발하는 비용보다 유지 보수하는 비용이 비교도 되지 않을 만큼 많이 발생한다.

## Good SW의 기준 및 attribute

- maintainability  
SW에 있어서 change는 피할 수 없다. component 간의 dependability, interaction을 잘 파악할 수 있어야 하고 해당 requirement가 왜 필요한지, 해당 부분을 왜 이런 방식으로 개발했는지 등이 잘 정의된 SW가 maintainability가 높다고 할 수 있다.
- dependability & security  
reliability, security, safety 등을 포괄적으로 포함한다. 해당 SW를 사용자가 원할 때마다 문제 없이 사용할 수 있는지, error가 자주 발생하지는 않는지 등을 포함하는 척도이다. 이 attribute를 높이기 위해 backup을 위한 불필요한 부분을 추가 한다든지, 사용자에게 여러 번 의도를 물어보면서 user error를 줄이도록 노력한다든지 등의 행위를 할 수 있다.
- efficiency  
SW 내에 필요 없는 부분은 없는지 등을 체크한다. responsiveness, processing time, memory 활용도 등 또한 포함된다.
- acceptability  
understandably, usable, compatible한 SW가 acceptable하다. 즉 SW를 사용하기 쉽고 이해하기 쉬워야 한다.

## General issue affect SW

- Heterogeneity  
Distributed System으로 동작해야 하는 경우가 빈번해지고 있다. 이 때 distributed system은 network로 연결이 되고 PC 뿐 아니라 mobile 등 다양한 기기에서 동작해야 한다.
- Business & Society  
Business와 Society는 매우 빠르게 변한다. 새로운 economy가 발견되기도 하고 새로운 기술이 개발되기도 한다. SW는 이러한 변화에 발 빠르게 대처해야 한다.
- Security & Trust  
SW가 실생활 곳곳에 사용되고 있는 만큼 security가 중요해졌고 해당 SW에 대한 신뢰도가 높아져야만 한다.

## Application types

- stand-alone  
네트워크의 연결이 없어도 사용할 수 있는 프로그램으로 한 프로그램 내에 모든 function이 포함되어 있다. PC에서 주로 사용한다. 예시로 MS Word 등이 있다.
- Interactive transaction-based  
원격 컴퓨터에서 실행시킬 수 있는 프로그램으로 web application이 대표적이다.예로 쇼핑몰 페이지 등이 있다.
- Embedded control  
SW control system, HW control system 등이 포함되고 SW 중 가장 많은 수를 차지한다. 예시로 smart-TV 등이 있다.
- Batch processing  
독립적인 input과 output을 처리할 수 있는 SW로 data 처리에 유리하다. salary pay 프로그램이 예시이다.
- Entertainment  
game 등 말 그대로 재미 위주로 개발된 SW이다.
- Modeling & Simulation  
과학 혹은 공학 중 특정 분야를 모델링하여 SW로 시뮬레이션을 진행하도록 하는 SW이다. 예시로 회로 설계에 사용되는 Pspice 등이 있다.
- Data Collection  
데이터 수집, process, 모으는 데 사용되는 SW로 IoT 환경 등 센서에서 데이터를 수집하는데 주로 사용된다. 예시로는 Magpi 등이 있다.
- System of systems  
시스템의 크기가 클 때 한 시스템 내 다른 세부 시스템이 포함될 수 있다. 이러한 유형을 일컫는다.

## SW Engineering with Web

- Web을 사용함으로써 application function을 web에서 접근할 수 있게 되었다. Web browser에 따라 UI가 바뀌는 것이 특징이다.
- web을 통해 개발을 진행함으로써 cloud computing이 가능하게 되었고 SW reuse가 활성화되었다. reuse 빈도가 높아짐에 따라 incrementally develop & deliver의 특징을 갖게 된다.

## Ethics

- SW가 실생활에 두루 사용되고 있는 만큼 개발자가 단지 법을 지키는 것을 넘어 강도 있는 양심적, 도덕적 기준을 지키는 것이 필요하다.
- Professional Responsibility  
회사 및 고객의 정보에 대한 기밀 유지(Confidentiality), 자신의 업무 능력에 맞는 일 선택(Competence), 고용주와 고객의 지적재산권 존중(Intellectual property rights), 컴퓨터 사적 사용 지양(computer misuse)
- ACM/IEEE code of ethics 존재  
8가지 규칙이 존재하고 규칙의 우선 순위가 존재한다.

이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다.
