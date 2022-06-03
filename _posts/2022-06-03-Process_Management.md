---
title: "2 Process Management"
excerpt: "process란, context, process state, interrupt, System call, IPC"
date: 2022-06-03
categories:
  - OS
tags:
  - Theory
  - OS
---
## Process

- kernel에 register된 실행할 entity, 실행(execute)되고 있는 프로그램(CPU, IO 사용, 대기 등 하는 상태)  
cf) run은 실제로 CPU를 사용하고 있는 상태이다.
- request를 보낼 수 있고 resource를 보낼 수 있는 상태이다.
- Process Control Block을 할당 받은 상태이다.
- Process는 active entity로 passive entity인 resource를 request, allocate, release 할 수 있다.
- 저장공간에 있던 application(executable code)이 실행되기 위해 Computer System에 submit (system boundary내로 들어간다, 만약 application이 종료되면 process가 소멸되고 나간다) → kernel에 register되고 process가 생성된다
- thread, task와 비슷한 개념이다.
    - thread
        - 프로세스 안에서 실행되는 흐름의 단위이다. 기본적으로 process는 하나 이상의 thread를 가진다. 프로세스 내에서 thread간 메모리와 공유하여 사용한다. stack만 따로 할당 받는다.
        - multithread일 때 각 CPU마다 하나의 thread를 맡아 실행함으로써 실행 속도를 높일 수 있다. 그러나 어떤 thread가 먼저 실행될지 알 수 없다는 단점이 있다. 따라서 동기화 문제가 발생할 수 있다.
    - process
        - 프로세스마다 하나의 PCB를 가진다. PCB는 kernel내에 생성되며 kernel이 관리하게 된다.
        - multiprocessing는 두 개 이상의 CPU가 하나 이상의 task를 동시에 처리하는 것이다. 다른 process로 변경할 때 context switching이 필요하고 이로 인해 overhead가 발생한다.

## Process Context

- text: program code
- data: global data
- stack: temporary data, local variable, function parameter, return address가 저장됨
- heap: execution 중 동적 할당되는 부분
- process registers value: PC, register 값 저장하는 부분

## Process Control Block

- PCB는 OS마다 구조체 등의 형태로 관리하게 된다.
- 시스템 성능에 있어 PCB 접근 속도는 중요한 지표이다.

### PCB에 포함된 정보

OS마다 포함된 정보는 다르나 대체적으로 아래 정보를 포함한다.

- PID
- Process state: 현재 process가 무엇을 하고 있는지
- Scheduling information: 주로 수치로 표현된 우선순위 정보가 저장됨
- Memory management information: memory의 어느 부분을 쓰고 있는지에 관한 정보, base register, page table, segment table등이 포함됨
- I/O status information: 어떤 I/O를 사용하고 있는지에 대한 정보
- Accounting information: 지금까지 CPU, IO 얼마나 사용했는지
- Context save area: register 값 임시 저장하는 공간

## Process state

<p align="center">
  <img src="/assets/images/process_state.png" alt="process state">
</p>


### create

- unix 에서 fork()
- transient state
- process가 만들어짐: PID, PCB 등을 부여 받음
- available memory check한 후 memory에 공간이 있다면 ready, 없다면 sspended ready가 됨

### ready

- create에서 memory가 충분하다면 ready 상태가 됨
- CPU를 할당 받으면 running 상태가 됨

### run

- CPU에서 실제로 실행되고 있는 상태
- ready와 run이 번갈아가면서 진행되며 context switching이 발생됨

### sleep

- I/O를 사용하기 위해서 스스로 CPU를 반납한 상태
- system call로 request
- interrupt를 기다리고 있는 상태
- I/O 마치면 바로 CPU로 돌아가는 게 아니라 ready 상태가 됨(ready 상태인 process만 CPU schedular가 CPU에 올리므로): wake up

### suspended sleep

- sleep 상태에서 메모리 부족으로 swap out 된 상태
- wake up되면 suspended ready가 됨
- swap in 되면 sleep 상태가 됨

### suspended ready

- create에서 memory 공간이 부족하다면 suspended ready가 됨
- ready에서 memory를 빼앗겨도(swap out, suspend)되어도 suspended ready가 됨

### terminate

- transient state
- running 후 실행이 완료된 상태
- PCB에 일부 정보만 남기고 반납함
- unix에서 exit()

## Scheduling

### Ready Queue

- ready state에 있는 process의 리스트
- process scheduling이 이 중 발생된다.

### I/O Queue

- sleep state에 있는 process: 각각 서로 다른 resource를 요청 중
- resource마다 다른 queue가 존재한다.

### Scheduler 종류

- long-term: job scheduler와 동일, multiprogramming을 관리하며 process를 어떻게 조합 할지를 결정한다
- medium-term: swapping을 결정한다.
- short-term: process scheduler, throughput, response time을 고려하여 process를 관리한다.

## Interrupt

- CPU 기준 외부(I/O device 등)에서 들어오는 unexpected event
- I/O, clock, console 등 다양한 interrupt가 존재한다.
- process가 system call을 하여 발생함

### Interrupt handling

interrupt → process exe 중지 → interrupt handling(interrupt 원인 파악 → interrupt service 결정 → Interrupt Service Routine)

- interrupt handling은 kernel 내 interrupt handler에 의해 관리됨  
cf) exception은 exception handling으로 처리함
- interrupt service: 지금 하는 일과 interrupt 중 우선순위 파악
    - 만약 interrupt가 더 시급하다면 → ISR
- interrupt handling  
interrupt → context saving → interrupt handling by interrupt handler → interrupt service → 다시 ready state → context restoring

## System Call

user mode에서 application이 resource를 할당받기 위해 system call을 사용한다. → OS가 kernel mode로 변경되며 해당 요청을 처리한다. 

process -(system call) → OS -(I/O request)→ I/O device -(Interrupt)→ OS -(wake up)→ process

### fork()

fork() system call을 이용하여 kernel이 memory에 application을 load 한다.
→ kernel은 fork()를 이용하여 부모 process의 heap, stack, data segment를 똑같이 갖는 child process를 만든다.

이 때 child process에는 새로운 pid가 부여된다.

### execve()

 만약 child process가 동일한 메모리를 갖고 싶지 않다면 execve()를 사용하여 새로운 프로그램을 실행할 수 있다.

### waitpid()

해당 pid가 끝날 때까지 기다린다

→ child process가 끝날 때까지 parent process에서 기다리는 형식으로 사용 된다.

## Context

### Context 종류

- user level: text, data, stack, heap (memory 내부)
- system level: PCB (memory 내부)
- register: register 값 (CPU 내부)

따라서 context 중 process 변경할 때는 register context만 저장했다 다시 불러오면 된다!

### Context Switching

- context saving:  process가 CPU에서 나올 때 register context 저장
- context restoring: register context를 다시 CPU register에 넣음
- context switching: process switching할 때 발생, 현재 process의 context를 save하고 다음 process의 context를 restore한다.
- PCB내에서 context save area가 context saving한 데이터를 저장하는 곳이다.
- context switching은 pure overhead이기 때문에 시스템 성능에 많은 영향을 미친다. overhead는 hw에 의해 결정된다.
    - CPU마다 구현 방식이 다르다.
    - 예로 current register set의 pointer를 변경하는 방식 등이 있다.

## Inter Process Communication

- information sharing, computation speed up, modularity, convenience등의 이유로 process간 통신이 필요하다.
- 특히 성능을 위해 한 application를 여러 process가 실행할 때 필요하다.
- 크게 message passing, shared memory 2가지 기술이 사용된다.
- 분산된 시스템에서 message passing, Remote Procedure Call, Distributed Shared Memory를 사용한다.  
cf) remote procedure call: 네트워크로 연결된 분산된 시스템에서 procedure call을 진행하는 방식이다. 주로 message passing 프레임워크의 가장 상단에 위치한다. client machine에서 server machine의 procedure 를 call 할 수 있다.  
Distribued Shared Memory: message passing 소통 시스템의 상단에 위치하는 software layer이다. OS kernel이나 runtime library routine으로 구현가능하다.
- Multi process 간에는 concurrency, synchronization 기술이 필요하다.

### 종류

- Message passing: send(), ack()를 통해 데이터를 주고 받는다. ack() 없이 일방적으로 데이터를 줄 수도 있다.
- Shared memory: read(), write()를 통해 공유되는 메모리에 데이터를 읽고 쓴다.
- Distributed Shared Memory(DSM): 각 machine이 쓰는 메모리 내에 각  process의 자리가 있어 다른 machine에 있는 본인의 자리에 데이터를 읽고 쓰면 해당 분이 공유 메모리처럼 사용된다. 네트워크로 연결된 시스템에서도 shared memory 개념을 사용한다.  

이 글은 공부한 내용을 정리한 내용으로 오류가 있을 수 있습니다. 오류가 있을 시 

