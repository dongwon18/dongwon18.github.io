---
title: "9 SW Testing"
excerpt: ""
date: 2022-01-20
categories:
  - SWEngineering
tags:
  - Theory
  - SWEngineering
---

안녕하세요.   
이 글은 소프트웨어공학개론 내용 정리 카테고리의 아홉 번째 글입니다. 다른 내용도 확인하고 싶다면 [introduction page](https://dongwon18.github.io/swengineering/SWEngineering_start/)를 확인해주세요.

- SW Testing 단계에서는 **intend & program defect** 를 확인합니다.
- error, anomaly, non-functional requirement, attribute를 확인 가능합니다.
- 명세서와 비교하는 verification & 요구사항과 비교하는 validation을 진행(V&V)합니다.
- validation testing: 요구사항을 만족하는가? test case, intend대로 동작하는가?
- defect testing: 잘못 동작하는 상황은 어떤 상황인가?

## Inspection

- review, static verification, source representation
- 장점
    - executable code 필요 없이 error 찾기
    - standard를 만족하는지 확인
    - portability, maintainability 체크
    - interaction 고려 안함
    - 다른 error에 가려지지 않음
- 한계
    - 고객의 real requirement 만족했는지 체크 불가
    - 일부 non-functional requirement 체크 불가(performance, usability 등)
- design test case -(test case)→ prepare data -(test data)→ run program with test data -(test result)→ compare result to test case →(test report)

## Stage of Testing

Development, release, user 3 단계로 진행되고 각 단계 내 세부 단계가 있다.

### Development

develop team이 진행하는 test로 unit test, component test, system test 순서로 진행된다.

- Unit Testing
    - detect testing의 한 종류로 unit을 테스트한다.
    - unit의 종류: function, method, object class, composite component
    - object class testing: 모든 operation, object attribute, possible state 테스트, inheritance가 테스트를 힘들게 함
    - automated testing: unit test를 자동화하는 것이 바람직함.
    - test case 고르기: requirement를 만족하는지 보여줘야 함, normal/abnormal operation 상황 포함
    - Partition testing
        - 모든 경우를 테스트하는 것은 불가능하므로 대표적인 케이스와 데이터에 대해 테스트를 진행하는 것을 말함
        - partition마다 test case 를 정해야 함
        - single value만 있는 sequence, sequence size 변화, sequence의 처음/중간/마지막 element test에 사용, 모든 에러 메시지를 만드는 경우, input buffer overflow, same input 반복, invalid output 만드는 경우, 계산 결과 너무 크거나 작은 경우
- Component Testing
    - interface(parameter, procedure massage passing, shared memory)를 테스트한다.
    - extreme end, null pointer, stress testing, read/write order 변경 등
- System Testing
    - interaction between component
    - compatible, 올바른 interact, 정확한 데이터를 transfer, emergent behavior 테스트 등
    - testing team이 따로 존재하기도 함

### Use Case Testing

- system testing의 base가 될 수 있음
- sequence diagram, use case diagram 사용하여 진행
- menu로 접근, function combination, user input 있는 곳 테스트 등

### Release Testing

특정 버전, release를 테스트 한다.

- developing team이 진행하지 않는다.
- 해당 버전이 사용하기 충분히 좋다는 것을 소비자에게 확신시키기 위해 사용한다. 따라서 functionality, performance, dependability, normal use에서 fail이 발생하지 않음을 보여야 한다.
- black box testing으로 어떻게 구현하는지보다는 specification에서 명시된 대로 동작하는지를 파악한다.
- system testing의 일종으로 볼 수도 있다. (system testing은 주로 defect, release testing은 validation testing)
- requirement based testing(requirement 검사, test 개발, test 진행), feature tested by scenario, performance testing(stress testing, emergent property 테스트) 등 진행

### User Testing

- 실제 사용자가 input을 제공하는 테스트로 시스템에 대한 조언을 주기도 한다.
- α, β, acceptance testing 진행
- user의 환경이 모두 다르므로 reliability, performance, usability, robustness 파악하기 위해 필수적인 테스트이다.
- acceptance test는 customized system에서 사용한다.  
define acceptance criteria -(test criteria)→ plan accept testing -(test plan)→ drive accept testing -(test)→ run accept testing -(test result)→ negotiate result -(test report)→ accept/reject system
- Agile 내에서 user testing
    - user가 팀의 일부이기 때문에 분리된 acceptance testing은 진행하지 않음

### TDD

- executable code보다 test code를 먼저 작성한다.
- incremental development, Agile에서 사용한다.
- plan driven에서도 사용 가능하다.
- identify new functionality → write test → run test → implement function & refactoring 반복
- 장점
    - code coverage 상승  
    exe code보다 test code를 먼저 작성하므로 놓치는 부분이 적다.
    - regression test  
    automated test를 하면서 어느 부분에서 test를 통과 못하는지 파악하기 쉽다.
    - simplified debugging  
    위의 이유로 인해 디버깅도 쉬워진다.
    - system documentation  
    test 자체가 document의 일환이므로 문서화에 도움이 된다.
 
이 글은 공부한 내용을 정리한 글으로 오류가 존재할 수 있습니다. 오류를 발견하신다면 댓글로 알려주시길 바랍니다
