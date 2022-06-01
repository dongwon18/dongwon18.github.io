---
title: "1 Operating System Introduction"
excerpt: "OS란, OS의 역할, OS 분류, Dual Mode Operation"
date: 2022-06-01
categories:
  - OS
tags:
  - Theory
  - OS
---

## Definition of OS

- application에게 system call을 이용해 HW 및 리소스를 사용할 수 있도록 하는 System SW Interface이다.
- application은 guest, OS는 host에 비유할 수 있다.
- OS는 하드웨어와 application의 사이에 위치한다.

## Function of OS

1. Process Management  
    System call을 이용하여 Process를 create, terminate, preempt, wake up, synchronization, deadlock handling 등을 진행한다.    
2. Resource Management      
    - HW: processor, memory, I/O device, network device 등을 관리
    - SW: files, message, processes 등 관리
    
3. Network/Security & Protection      
    trasmission layer(TCP), network layer(IP)를 OS에서 담당한다.    
4. User Interface  
사용자 인터페이스를 제공한다.
    - Command Line Interface: 사용자가 실행 명령, 파일 이름을 기억하고 있어야 사용 가능한 초기에 사용됐던 인터페이스
    - Graphical User Interface: 현재 사용하고 있는 click 형태의 인터페이스
    - End-User Comfortable Interface: 음성 인식, 제스처 등을 기반으로 하는 인터페이스, 별도 장비 필요

## OS Classification

### 사용자 수 따라
- single user      
    한 번에 한 명의 사용자만 존재하기 때문에 간단하다. MS DOS, PC용 OS, micro computer의 OS가 대부분 이에 해당한다.
- multiple user      
    한 번에 두 명 이상의 사용자가 사용한다. 따라서 보안 및 파일 보호 기능이 뛰어난 OS가 필요하다. Unix, Linux가 이에 해당한다.
      
### Process 수 따라
- single tasking process      
    한 번에 한 프로세스만 실행하기 때문에 간단하다. 따라서 multi-user OS에서는 사용될 수 없다. MS DOs가 예시이다.    
- multi-programming, multi-tasking      
    한 번에 두 개 이상의 프로세스를 실행한다. cuncurrency 제어, synchronization between process 기술이 필요하기 때문에 single tasking process보다 복잡하다.
    
### 역할에 따라
batch, Interactive, real-time, PC, Embedded, Could, Mobile 등 다양하다.

## Dual Mode Operation
- OS는 **kernal mode, user mode** 2개의 모드로 동작한다. 만약 interrupt나 trap이 발생했을 때 user mode에서 kernel mode로 변경한다. 
- kernel mode일 때만 사용 가능한 priviledged Instruction이 존재한다.   
  - ex) timer, create, interrupt handler, I/O control, switch to user mode 등
