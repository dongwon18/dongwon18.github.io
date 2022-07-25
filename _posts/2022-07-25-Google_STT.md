---
title: "Google STT 사전 조사"
excerpt: "Google Speech To Text 특징 알아보기, 예제 실행하기"
date: 2022-07-25
categories:
  - VoiceRecognition
tags:
  - Tutorial
  - RPI
  - Google_STT
  - example_code
---
Raspberry Pi에서 Google STT, picovoice Porcupine 등을 이용하여 간단한 한국어 음성 인식이 가능한 프로젝트를 진행하려고 합니다.

이 글은 프로젝트에서 사용되는 Google Speech To Text를 사전 조사한 내용을 포함하고 있습니다. 

## STT 3가지 방식

- 동기 인식: STT API로 음성 데이터를 보내고 처리된 후 결과 확인, 1분 이하의 오디오 데이터에 한해서만 가능
- 비동기 인식: STT API에 음성 데이터 보내고 인식 결과를 주기적으로 폴링 가능, 480분 길이 가능
- 스트리밍 인식: 실시간 인식 용도, 오디오 캡처 중에도 결과 제공

## 동기 방식

- stt 끝나고 나야 다음 일 가능
- 평균 30초 길이 오디오 파일을 처리하는데 15초 걸림
- 오디오 품질이 나쁘면 오래 걸림
- REST API, gRPC 방식 존재
- 결과 좋으려면 FLAC, LINEAR16 사용
- config 파일에 WAV, FLAC 같은 경우 encoding, sampleRateHertz는 안 써도 됨
    - [speechContext](https://cloud.google.com/speech-to-text/docs/context-strength?hl=ko#speech-context-strength-python)에 phrase를 넣어서 음성 인식 작업의 힌트를 제공하는 단어, 구문 목록을 넣을 수 있음
- audio
    - json 직렬화와 호환, base64 인코딩
    - 가능하다면 16000Hz 가 좋음, 낮으면 성능 저하 높아도 성능 향상되지 않음
        - 이미 되어 있는 걸 16000Hz로 변경하는 건 권장하지 않음
- AI model 정해서 결과 받을 수 있음
    - latest_long: 미디어, 대화 등 긴 콘텐츠
    - latest_short: 짧은 발화, 명령어
    - video: 여러 명의 화자, 배경 소음 많을 때, 마이크 성능 좋을 때(프리미엄 모델)
    - phone_call: 전화 통화, 8000Hz(프리미엄 모델)
    - command_and_search: 짧은 오디오 클립
    - default: 위의 예시 외
    - medical_dictation: 의료 전문과 메모
    - medical_conversation: 의료 전문가와 환자 간 대화
    - 한국어는 `command_and_search, default, latest_long, latest_short` 가능

## 가격

- free tier: 월별 60분 무료
- 100만분까지 15초 당 $0.006 달러(로깅 제외, 기본) $0.004(로깅 포함), 15초 단위로 올림으로 계산
    - 로깅: 구글에 데이터를 제공하는 것

## 예제

- anaconda 이용 진행
- `Google Speech-to-Text console 간단 사용법` 따라 project 세팅 및 json auth 파일 받기

1. [authentication 등록](https://cloud.google.com/docs/authentication/getting-started#windows)
    - cmd에서 진행하는 것처럼 진행하니까 잘 됨
2. samples/snippets/requirements.txt 로 환경 세팅
    - anaconda3 google_stt 라는 가상 환경 만들어서 진행
3. [quickstart.py](http://quickstart.py) 실행
4. 결과
```
Transcript: how old is the Brooklyn Bridge
```
## 참조

- [Google Speech-to-Text DOCS](https://cloud.google.com/speech-to-text/docs/basics?hl=ko)
- [Google Speech-to-Text console 간단 사용법](https://cloud.google.com/speech-to-text/docs/transcribe-console?hl=ko)
- [Google Speech-to-Text python guide](https://github.com/googleapis/python-speech/tree/main/samples/snippets)
