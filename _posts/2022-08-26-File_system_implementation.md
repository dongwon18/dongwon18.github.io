---
title: "9 File System Implementation"
excerpt: "Layered, Disk"
date: 2022-08-26
categories:
  - OS
tags:
  - Theory
  - OS
---

## Layered file system

application program ↔ logical file system ,↔ file organization module ↔ basic file system ↔IO control ↔ device

- kernel은 이 중 logical file system() ~ basic file system 관리
- logical file system: file system의 metadata, FCB 관리, protection, security
- file organization module: logical block addr , physical 을 연결, free space mngm
- basic file system: device driver와 상호작용
- IO control: device driver, interrupt handler

## Disk layout

- boot control block
- volume control block: 쓰고 있는 block 개수, block 크기, 빈 블록 개수,  pointer 등
- FCB
- data blocks

## Disk Allocation Method

### contiguous

- 파일을 연속적으로 할당
- space 효율성 떨어짐
    - 새로운 파일의 자리 찾기 힘듦: best fit, first fit 등 이용
    - external framentation
    - 파일마다 얼마나 자리 차지할지 모름

### linked

- linked list로 블록이 연결됨
- 다음 블록 주소를 가지고 있음
- external fragmentation 없음
- 단점
    - direct-access하기 힘듦
    - pointer를 위한 자리 필요함
    - reliability: 링크 하나 끊기면 다음 거 찾을 수 없음
- FAT

### indexed allocation

- index block에 모든 pointer 모음 → direct access 가능해짐
- external fragementation 없음
- 파일마다 index block이 따로 존재
- 단점
    - index block이 자리를 차지함
- Unix, linux 사용  

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하신다면 댓글로 알려주세요!
