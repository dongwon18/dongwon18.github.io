---
title: "8 Virtual Memory Management"
excerpt: "flat, 2 level, hierarchical, general graph"
date: 2022-08-10
categories:
  - OS
tags:
  - Theory
  - OS
---
## Directory Structure

### Flat

- 단점
    - 파일이름이 모두 달라야 함
    - multi-user 파일을 구분 힘듦
- 장점
    - system mngm가 용이

### 2-level

- 장점
    - multi-user의 파일 구분 가능
    - 이름 겹쳐도 됨
    - 사용자마다 각자 user file directory를 따로 사용 가능
- 단점
    - user간 cooperation 지원 안 함
    - system file, shared file 어디에 놔야 할지 애매함

### Hierarchical(acyclic graph)

- home directory, current directory, absolute path relative path 등의 용어가 생김
- 장점
    - 사용자가 각자 subdirectory 만들 수 있음
- Unix에서 link를 이용해서 구현 됨

### General graph(cycle 가능)

- 단점
    - 파일 찾기할 때 무한 루프
    - 삭제 시 self referencing 발생

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주신다면 감사하겠습니다.
