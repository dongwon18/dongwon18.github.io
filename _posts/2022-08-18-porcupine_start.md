---
title: "Porcupine 시작하기"
excerpt: "picovoice, porcupine, hot word, wakeup word, 예제 실행"
date: 2022-08-18
categories:
  - VoiceRecognition
tags:
  - Tutorial
  - RPI
  - Picovoice
  - Porcupine
  - hot_word
  - example_code
---
이 글은 [Raspberry Pi, Google STT를 이용한 한국어 음성 인식 프로젝트](https://dongwon18.github.io/categories/#voicerecognition)
의 일부입니다.

안녕하세요. [이전 글](https://dongwon18.github.io/voicerecognition/stt_services/)에서 정한대로 picovoice 사의 porcupine을 이용하여 wake up word 기술을 사용해보려고 합니다.

## Wake up word

그 전에 간단하게 wake up word가 어떤 것인지 알아보려고 합니다.

- 사용 이유
    - wakeup word 이후 말한 것만 녹음함으로써 사생활 존중
    - 항상 녹음을 하고 분석하면 시스템에 부하가 많이 가므로 wake up word가 인식되면 그 이후에 녹음되는 것부터 stt를 돌림으로써 시스템 부하를 줄임
    - wake up word 인지 후 stt 실행하는 건 실제로 시리 등 음성 비서가 사용하는 기본적인 구조
- wakeup word가 되는 조건
    - 간단한 단어 + 브랜드 이름을 붙임으로써 마케팅 효과(예시 ok Google)
    - 다른 언어 사용되어도 동일한 발음을 하는 단어를 선택(uniformity)
    - false positive를 고려하여 평소에 별로 사용하지 않는 단어 조합을 골라야 함

## Porcupine demo 사용해보기

Porcupine은 다양한 환경에서 사용 가능하지만 저는 google stt와 연결하는 걸 고려해서 python으로 사용했습니다. dependency를 고려하여 [google stt RPI에 설치 글](https://dongwon18.github.io/voicerecognition/Install_Google_STT_in_RPI/)에서 만들었던 가상 환경에서 진행했습니다.

- google stt를 설치했던 프로젝트 폴더로 이동하여 가상 환경 실행
`source env/bin/activate`
- 패키지 설치
`pip3 install pvporcupine`
- 데모 설치
`pip3 install pvporcupinedemo`
- picovoice에서 access key 얻기
    - picovoice 로그인
    - `profile`에서 `Show AccessKey` 에서 key 얻기
- picovoice console에서 custom wake up word 만들기
    - [picovoice console](https://console.picovoice.ai/ppn) 로 이동
    - 언어, platform을 선택하고 phrase에 원하는 wake up word 작성하기
    - `Train Wake Word`를 클릭해 키워드 학습 시키기
- keyword 파일과 한국어 파일 라즈베리파이에 저장하기
    - 프로젝트 폴더에 두 파일을 저장
- 데모 파일 실행하기
    - `porcupine_demo_mic --access_key [access key] --keyword_paths [프로젝트 폴더]/hi_bobi_ko_rpi.ppn --model_path [프로젝트 폴더]/porcupine_params_ko.pv` 실행
    - 한국어로 진행하는 예제
    [여기](https://github.com/Picovoice/porcupine/tree/master/lib/common) 에 영어, 독일어, 스페인어, 프랑스어, 이탈리아어, 일본어, 한국어, 포르투갈어 언어 모델이 있으므로 필요에 따라 선택해서 사용

## 결과

이와 같이 실행하면 원하는 hot word를 인식할 수 있습니다. 다음에는 demo 코드를 이용하여 google stt와 연결하는 부분을 진행해 보겠습니다.

## 참조

- [picovoice porcupine demo](https://github.com/Picovoice/porcupine/tree/master/demo/python)
- [porcupine get started](https://picovoice.ai/docs/quick-start/porcupine-python/)
