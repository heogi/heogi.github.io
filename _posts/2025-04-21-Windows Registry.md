---
title: Windows Registry for Forensics
author: heogi
date: 2025-04-21
categories:
  - forensics
  - windows
tags:
  - forensics
  - windows
comments: true
---
## Windows Registry 구조
### Keys and Values
* `Registry Keys` - 폴더 처럼 서브 키를 가질 수 있는 경로
* `Registry Values` - 키 아래에 저장되는 데이터(데이터 이름 + 형식 + 실제 값)
* `Registry Hives` - 디스크에 저장되는 Registry 데이터 파일

### Root  Keys

| Root Key                    | Description                                    |
| --------------------------- | ---------------------------------------------- |
| HKEY_CURRENT_USER(`HKCU`)   | 현재 로그온한 유저의 프로필 설정 (Desktop, 제어판, 기타 유저 종속 설정) |
| HKEY_USERS(`HKU`)           | 모든 유저의 프로필 설정                                  |
| HKEY_LOCAL_MACHINE(`HKLM`)  | 시스템 설정 (Hadware, Software 등)                   |
| HKEY_CLASSES_ROOT(`HKCR`)   | 윈도우에서 사용하는 파일 정보 (특정 확장자에 대해 어떤 프로그램으로 실행할지 등) |
| HKEY_CURRENT_CONFIG(`HKCC`) | 시스템 실행 시 사용할 하드웨어 프로필 설정                       |

