---
title: "Wake up word, STT 제품 비교"
excerpt: "Google Speech To Text 특징 알아보기, 예제 실행하기"
date: 2022-08-08
categories:
  - VoiceRecognition
tags:
  - Tutorial
  - RPI
  - hot_word
  - wakeup_word
---


이 글은 [Raspberry Pi, Google STT를 이용한 한국어 음성 인식 프로젝트](https://dongwon18.github.io/categories/#voicerecognition)의 일부입니다.  

안녕하세요. 
이번에는 프로젝트에서 사용할 STT, wake up word 제품을 어떤 걸 사용할지 정해보겠습니다.

## Wake up word 개념

- [VIVOKA](https://vivoka.com/google-speech-wake-up-word/) 참고
    - trigger the voice recording of the user의 기능
    - 사용 이유
        - wakeup word 이후 말한 것만 녹음함으로써 사생활 존중
        - 항상 녹음을 하고 분석하면 시스템에 부하가 많이 가므로 시스템 최적화
    - wakeup word가 되는 조건
        - 간단한 단어 + 브랜드 이름을 붙임으로써 마케팅 효과(ok Google)
        - 다른 언어 사용되어도 동일한 발음을 하는 단어를 선택(uniformity)
        - false positive를 고려하여 평소에 별로 사용하지 않는 단어 조합을 골라야 함

## STT 제품
- picovoice의 leopard
    - 영어만…되는 듯?
    - 100 hour/달 free tier
    - leopard는 음성 데이터 cheetah는 streaming 음성 데이터
    - 오프라인에서 돌아감
- 카카오 stt
    - 음성 인식 20000건, 90분/하루
    - 2021년 7월 1일부로 서비스 종료
- naver stt
    - 프리티어가 없음
        - 15초 당 4원

## Wake-up word 제품

- picovoice의 [porcupine](https://github.com/Picovoice/porcupine)
    - 한국어 지원, 라즈베리파이에서 동작 가능, 아파치 라이센스
    - star 2700
    - 97% 이상의 정확도
    - 단어 고르는 [팁](https://picovoice.ai/docs/tips/choosing-a-wake-word/), 단어에 따라 정확도 차이 남
    - RPI3 기준 memory 1MB, single core 4% 미만으로 사용
    - 특정 사람의 목소리가 아니라 범용 사람 목소리에 반응
    - wake up word 사용 전에 굳이 조용히 하고 있다가 단어를 얘기할 필요는 없음
    - 전력 소모는 HW에 따라 달라짐
    - 비슷한 억양의 단어를 분별하는 기능은 없음(악센트 등 때문에), 정확도 높이면 false acceptance는 줄이는 대신 miss rate는 높아짐
    - offline에서도 가능
    - wakeup은 무료
    - youtube [예시](https://www.youtube.com/watch?v=dgm91iCiEQo)
    - [starting guide](https://picovoice.ai/docs/porcupine/)
- snowboy
    - RPI 지원
    - 무료, 아파치 라이센스
    - 2700 star
    - 2020 12월 31일에 서비스 종료 해서 docs 등 아무것도 못 봄
    - 한국어 지원하는지도 모르겠음
- [pocketSphinx(CMUSphinx)](https://cmusphinx.github.io/)
    - language independant
        - 한국어는 미리 학습된 [모델이](https://sourceforge.net/projects/cmusphinx/files/Acoustic%20and%20Language%20Models/) 없어서 학습 직접 시켜야 함
    - 2022년 7월 23일에 버전 5 업데이트함…
        - 2019년에 서비스 종료 했다 다시 시작
    - 완전 무료
    - 3100 star

## 결론
- STT는 한국어 인식률이 좋고 접근성 또한 좋은 Google STT를 사용하기로 했습니다(naver도 한국어가 가능하지만 구글이 프리티어를 제공한다는 측면에서 더 매력적이었어요)
- Wake up word는 한국어 모델이 제공되고 무료인 picovoice의 porcupine 제품을 사용합니다.

더 궁금하신 사항이나 조언이 있으신다면 편하게 댓글로 알려주세요!
