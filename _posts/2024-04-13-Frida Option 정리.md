---
title: Frida Option 정리
author: heogi
date: 2022-07-10
categories:
  - Mobile
  - Frida
tags:
  - Frida
comments: true
---
```shell
[Device list]
frida-ls-devices

[기기 연결]
-U, --usb # USB로 연결된 기기 연결
-D, <Device ID> # 가상 기기 연결 / -D emulator-5554

[App Attach]
frida -F # Foreground에서 실행되고 있는 APP 연결
frida -n <Pakage Name> # APP의 패키지 이름으로 연결
frida -p <PID> # APP의 PID로 연결

[APP Spawn & Attach]
frida -f <Pakage Name> # APP을 실행하고 연결

[Load Script]
frida -l <Script file> # Frida Script 파일을 로드

[Process list]
frida-ps -a # 동작중인 APP 프로세스 목록 출력
frida-ps -i # 동작중이거나 설치된 APP 프로세스 목록 출력

[Module & Function Trace]
frida-trace -I <Module> # 모듈의 모든 함수 추적
frida-trace -X <Module> # 해당 모듈의 모든 함수를 추적에서 제외
frida-trace -i <function> # 함수 추적
frida-trace -x <function> # 해당 함수를 추적에서 제외
frdia-trace -d, --decorate # 추적할 함수가 속한 모듈이름을 함께 출력

[Kill]
frida-kill -D <Device ID> <PID> # 해당 PID의 프로세스 종료
```