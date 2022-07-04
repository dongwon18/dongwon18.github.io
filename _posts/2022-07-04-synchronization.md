---
title: "4 Synchronization"
excerpt: "scheduling goal, performance 척도, sheduling 기법"
date: 2022-07-04
categories:
  - OS
tags:
  - Theory
  - OS
---
## Synchronization이란

- 다른 process가 무엇을 하고 있는지에 대한 정보를 얻는 것
    - Process는 독립적이기 때문에 다른 process에 대한 정보 모름
    - 여러 process가 동시에 system내에 존재
- multi process system에서 필수

## Race Condition

- shared memory에 동시에 접근하려고 할 때 접근 순서에 따라 결과가 변경되는 상태
- ciritical section: 여러 process가 실행하는 code 부분
- ciritical section은 동시에 하나의 process만 들어갈 수 있도록 해야 함 → mutual exclusion
- 동시에 진입하지 못하도록(mutual exclusion), 아무도 없는데 진입할 수 없는 경우가 없도록(progress), 한 process만 계속적으로 막지 않도록(bounded wait) 해야 함

## Mutex

```c
// P0 Dekker's algo
do{
	flag[0]=TRUE;
	while(flag[1]){
		if(turn==1){
			flag[0]=FALSE;
			while(turn==1);
			 flag[0] = TRUE;
    }
		// critical section
  }
	turn = 1;
	flag[0]=FALSE;
}while(1);

// P0 Peterson's algo
do{
	flag[0]=TRUE;
	turn=1;
	while(flag[1] && turn==1);
	// critical section
	flag[0]=FALSE;
}while(1);

// N-process Pi Dijkstra
do{
	do{
		flag[i] = WANT_IN;
		while(turn != i){
			if(flag[turn]=IDLE) turn = i;
		}
		flag[i] = IC_CS;
		j = 0;
		while((j < n) && (j == i || flag[j] != IN_CS)) j = j+1;
	}while(j < n);
}while(1);
```

- flag, turn을 이용하여 critical section을 SW 적으로 관리
- 느리고 mutual execlusion 중 preemption이 발생할 수도 있다.
→ HW instruction 사용

## HW instruction

### Test and set

- lock 초기화: false로
- 지금 값 return 하고 값을 true로 변경하기

```c
bool TS(bool *target){
	bool rv = *target;
	*target = true;
	return rv;
}
```

### Compare and swap(CAS)

- lock 초기화 false로
- lock 값이 expected와 같으면 새로운 값으로 변경
- 지금 값 return
- bouded waiting 해결 가능

```c
void swap(bool *a bool *b){
	bool temp = *a;
	*a = *b;
	*b = temp;
}
```

- 요즘은 fetch and add, compare and swap을 주로 사용함

### 문제

- while loop를 돌면서 기다리고 있기 때문에 busy waiting을 함→ 효율 떨어짐
- busy waiting을 해결하기 위해 semaphore, sequencer/eventcount를 사용하게 됨

## Semaphore

- 3가지 operation: P(), V(), init
이 op은 indivisible로 중간에 interrupt 불가(즉 하나의 machine inst라는 뜻)
    - P(): wait(), sleep 상태로 갈 때 사용
    - V(): signal(), wake up 시킬 때 사용

```c
P(S)
if(S >0)
	then S <- S-1;
else wait on the queue Q;

V(S)
if(waiting process in Q 존재)
	then wakeup one of them;
else S <- S+1;

P(active)
# critical section
V(active)
```

- binary semaphore(ME, synchronization에 이용), couting semaphore(procuder-comsumer에 이용) 등 존재
- binary semaphore로는 ME, sync만 해결 가능
- producer-consumer, reader-writer, dining philosopher는 counting 사용해야 함

## Eventcount/Sequencer=ticket lock

- Sequencer: 티켓 뽑아간 사람 수
감소하지 않음, ticket() op로만 접근 가능
- ticket: 번호 부여, indivisible op
- Eventcount: 현재 순서인 번호
감소하지 않음, read(), advance(), await() op로만 접근 가능
- read: 현재 eventcount의 값을 반환
- advance: eventcount 증가, 갱신된 번호표에 해당하는 process 깨우기
- await: 순서 아닌 process waiting queue에 넣기

```c
v=ticket(S);
await(E, v);
# critical section
advance(E);
```

- counting semaphore가 해결하는 문제 다 해결 가능
- FIFO scheduling으로 진행됨 → starvation 없음
- dining 철학자 문제에서 semaphore보다 비효율적

이 글은 배운 내용을 정리한 것으로 오류가 있을 수 있습니다. 오류를 발견하셨다면 댓글로 알려주세요!

