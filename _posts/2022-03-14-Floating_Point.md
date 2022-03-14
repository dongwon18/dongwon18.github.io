---
title: "6 Floating Point"
excerpt: "IEEE FP"
date: 2022-03-14
categories:
  - ComputerArchitecture
tags:
  - Theory
  - ComputerArchitecture
---

안녕하세요.  
이 글은 컴퓨터구조론 카테고리의 여섯 번째 글입니다. 
다른 글도 궁금하시다면 [컴퓨터 구조론 시작하기](https://dongwon18.github.io/computerarchitecture/Computer_Architecture_Start/)를 확인해주세요.

## Floating Point
컴퓨터에서 실수를 나타내는 방식 중 하나이다.  
(다른 하나는 fixed point이다. 몇 비트까지는 정수부 나머지 부분은 실수부 식으로 소숫점의 자리가 고정되어 있는 방식이다.)
3 파트로 나뉨  
- sign bit: 1이면 음수, 0이면 양수이다.
- exponent: 2 제곱의 지수 부분이다. exponent를 unsigned로 나타내는 대표적인 방식으로  Excess-16이 있다. 이 방식에서는 지수 값에 10000<sub>2</sub> 를 더해 표현한다.
즉 실제 지수 값을 구하기 위해서는 exponent - 16을 진행해야 한다.
- fraction(= significand, mantissa): 2<sup>exponent</sup> x X에서 X

## IEEE FP

- Denormalized values
    - exp = 0
    - exponent value E = 1 - Bias
    - M = 0.xxxx(1이 앞에 없음)
- Normalized values
    - exp ≠ 0 & exp≠ 11111
    - exponent value = Exp - Bias  
    Exp: unsigned value denoted by exp  
    bias: 2<sup>k-1</sup> -1  
    - M = 1.xxxx  
    minumum: M = 1.0  
    maximum: M = 1.111111111 = 2.0 - ε  
    앞에 1이 있기 때문에 유효숫자를 하나 더 얻는 효과가 있다.
- Special values
    - exp = 11111
    - frac = 0이면 무한대 표현
    - frac ≠ 0이면 NaN

2003<sub>10</sub> = 111 1101 0011<sub>2</sub> = 1.1 11101 0011<sub>2</sub> * 2<sup>10</sup>  
M = 111 1101 0011<sub>2</sub>  
frac = 111 1010 0110 0000 0000 0000<sub>2</sub>  
E = 10  
Exp = E + Bias = 10 + 2<sup>8-1</sup> = 137 = 1000 1001<sub>2</sub>  

<p align="center">
  <img src="/assets/images/floating_point.png" alt="floating point는 불균일 간격이다"> <br/>
  이미지 출처: https://www.sciencedirect.com/topics/engineering/floating-point-number
</p>

결론적으로 floating point를 이용했을 때 값이 커질수록 표현하지 못하는 수가 더 많아진다. 또한 표현 가능한 수 사이의 간격이 일정하지 않다는 특징 또한 있다.
이러한 특성으로 인해 코딩 시 실수 변수를 사용할 때는 동일(==) 기호가 아니라 부등호를 사용하는 것이 바람직하다.  

이 글은 공부한 내용을 정리한 글로 오류가 있을 수 있습니다. 오류를 발견하셨다면 댓글로 알려주세요!
