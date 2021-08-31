---
title: "Android Studio 삭제하기(Uninstall Android Studio)"
excerpt: "uninstall.exe 없을 때 (uninstall.exe not found)"
date: 2021-08-31
categories:
  - ErrorHandling
tags:
  - ErrorHandling
  - Programming
  - AndroidStudio
  - Windows10
---

안녕하세요.   
안드로이드 스튜디오를 삭제하다가 특이한 에러를 겪어서 글을 남깁니다.  

<p align = 'center'>
  <img src = "/assets/images/uninstall.jpg" alt = 'Android Studio uninstall.exe not found'> <br/>
  uninstall.exe를 찾을 수 없습니다.
</p>


- 안드로이드 스튜디오 버전: Android Studio 2020.3.1.x(저는 2020.3.1.21과 2020.3.1.22 였습니다.) for Windows 10 (64bit)  
- 발생 날짜: 2021-08-31  
## 문제 상황
1. `제어판 -> 프로그램 제거` 에서 제거 버튼을 눌렀을 때 에러 메시지를 확인하게 됩니다.
`C:\Program Files\Android\Android Studio\uninstall.exe 을(를) 찾을 수 없습니다.` 
2. 실제로 해당 폴더에 가봤을 때 uninstall.exe를 찾을 수 없었습니다.
3. 검색을 해 본 결과 최근 버전에서 해당 문제가 발견되는 것 같았습니다.  
(reddit의 질문 글이 26일 전 업로드 되어 있었고 많은 분들이 동일한 문제를 겪고 계신 듯 했습니다.)  

## 해결 방법
해당 reddit 질문의 답변을 참고해 본 결과  
1. 이전 Android Studio 버전을 사용중인 사람은 uninstall.exe가 존재한다는 것을 알 수 있게 됐습니다.
2. 이전 버전을 설치하여 uninstall.exe를 삭제하고 싶은 문제 있는 버전의 경로에 놓고 uninstall.exe를 실행하면 해결됩니다.
(저는 얼마나 이전 버전을 설치해야 할지 애매해서 Android Studio 4.0.1 버전을 다운로드 했고 uninstall.exe가 잘 동작했습니다.)

## 참고
- [reddit.com의 질문 글](https://www.reddit.com/r/AndroidStudio/comments/oy7397/cant_uninstall_android_studio/)

<p align = 'center'> 이 글이 도움이 되셨다면 댓글로 알려주세요^^ </p>
