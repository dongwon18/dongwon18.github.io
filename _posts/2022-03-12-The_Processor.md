---
title: "3 The Processor"
excerpt: "CPU 하는 일, CPU 내부 구성, critical path"
date: 2022-03-12
categories:
  - ComputerArchitecture
tags:
  - Theory
  - ComputerArchitecture
  - MIPS
---
이 글은 컴퓨터구조론 카테고리의 세 번째 글입니다. 다른 글도 궁금하시다면 [컴퓨터 구조론 시작하기](https://dongwon18.github.io/computerarchitecture/Computer_Architecture_Start/)를 확인해주세요.

MIPS 32 bit 기준으로 정리하였습니다.

## CPU does

- fetch  
instruction을 메모리에 불러온다. 이 때 실행 중인 다음 instruction을 Process couter register(PC)에 저장한다.
- decode  
opcode field를 이용하여 instruction 종류를 판별한다.

자세히는

1. Arithmetic & Logical(add, sub, and , or, slt)  
rs, rt 2개의 word를 레지스터 파일로부터 읽는다   
→ 계산한다   
→ 결과를 레지스터 파일에 적는다(rd)   
→ PC 에 4를 더한다  
(현재 다루고 있는 MIPS는 32 bit register를 사용 중이기 때문에 다음 instruction은 4 word 뒤이다. branch, jump가 아니면 모두 +4로 다음 명령어로 넘어간다.)
2. Memory Access(lw) 읽기  
rs 1개의 word를 레지스터 파일로부터 읽는다   
→ base address를 더한다   
→ 데이터 메모리에서 word  
(방금 구한 주소 = base address + offset에 있는 데이터)를 불러온다  
→ 레지스터 파일 rt에 불러온 값을 적는다  
→ PC+4
3. Memory Access(sw) 쓰기  
rs 1개의 word를 레지스터 파일로부터 읽는다   
→ base address를 더한다   
→ 레지스터 파일 rt에 저장된 값을 읽는다   
→ word(방금 구한 주소 = base address + offset에 저장)를 데이터 메모리에 저장한다.  
→ PC+4
4. Control Transfer(beq) a == b 에 따른 조건문  
rs, rt 2개의 word를 레지스터 파일로부터 읽는다   
→ ALU를 이용해 비교 연산을 진행한다   
→ 만약 동일하면 PC = PC + 4 + offset(immediate field) * 4 로 설정하여 조건문을 수행한다 
    / 다르다면 그냥 PC + 4 
5. Control trasfer(j)  
PC = PC[31:28] | (address filed) * 4 로 설정한다.

## Instruction

### R format

**opcode[31:26] rs[25:21] rt[20:16] rd[15:11] shamt[10:6] func[5:0]**  
ex) add $t0,  $s1, $s2

<p align="center">
  <img src="./assets/images/CPU.drawio.png" alt="기본 Processor Architecture"> <br/>
  기본 Processor architecture
</p>

- 2개의 operand가 있음
- arithmetic, logical operation
- destination regiser 에 결과를 작성함

#### Register File

- Read  
combination circuit처럼 rs, rt register에 저장되어 있는 값(32bit)을 읽는다.  
32개의 register의 값을 주솟값으로 MUX로 골라 읽는다.
- Write  
sequential circuit처럼 동작하여 rd register에 쓸 데이터를 저장한다. 1 clock period가 걸린다. regwrite를 flag로 활용하여 regwrite가 high면 쓰기 모드, low면 읽기 모드로 동작한다.  
n-to-2<sup>2 decoder로 rd 값마다 할당된 register에 값을 저장한다.

#### ALU

- ALU Operation 4bit으로 어떤 연산을 할지 결정한다.
- 32bit operand 2개를 input으로 받아 operation을 수행하여 32bit output으로 내보낸다.

### I format

opcode[31:26] rs[25:21] rt[20:16] immediate[15:0]
ex) addi $t9, $zero, 100

- 1개의 base register address, dest or src register, constant or address 로 이뤄짐
- read, write
- constant or address 는 15:0까지를 차지한다. 그러나 주소는 32bit이므로 31:16까지는 sign bit으로 채운다.
- load: MemRead, MemWrite, RegWrite(101)  
store: (010)
- 하나의 ALU가 I format, R format 모두에 사용되므로 MUX를 이용해서 read data2를 사용할지 I type용 constant or address를 사용할지 선택한다. 선택은 ALUSrc를 이용하여 진행된다.
- ALU의 결과를 dst register에 저장해야 하는 Rtype과 구분하기 위해 ALU의 결과는 data memory에 저장되고 MUX를 이용하여 Memtoreg이 1일 때 dst register에 저장된다.

#### Branch (beq)

- PC+4에 branch address를 더해줘야 하기 때문에 PC+4 뒤 ADD를 위한 ALU를 추가한 후 MUX로 PC+4를 선택할지 PC+4+displacement * 4를 선택할지 결정한다.
- MUX의 선택은 ALU에서 0인지 판별한 값과 branch inst인지 판별하는 값을 AND 로 처리한다.

#### Jump

- instruction[25:0]을 left shift(*4와 동일) 하여 beq용 MUX 뒤에 jump용 MUX를 하나 더 설치하여 branch가 아니고 jump면 instruction[25:0] * 4를 다음 PC로 설정한다.

### Control Signal

RegDst ALUSrc MemtoReg RegWrite MemRead MemWrite Branch ALUOP[1] ALUOP[0]

R-format: 1 0 0 1 0 0 0 1 0
lw: 0 1 1 1 1 0 0 0 0 
sw: X 1 X 0 0 1 0 0 0
beq: X 0 X 0 0 0 1 0 1

<p align="center">
  <img src="./assets/images/CPU.jpg" alt="최종 architecture"> <br/>
  최종 architecture
</p>

## 정리

- critical path: load instruction
inst mem → register file → ALU → data memroy → register file
load는 매우 자주 쓰이는 inst임에도 불구하고 느리다. 즉 Amdahl’s law에 어긋난다. 따라서 pipelining을 이용하여 성능을 향상시킨다.
  
이 글은 배운 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하시면 댓글로 
  
  
