---
title: Windows Folder and Service 권한
author: heogi
date: 2025-05-07
categories:
  - Forensics
  - Windows
tags:
  - forensics
  - windows
comments: true
---
## 1. Windows Folder 권한
`icacls.exe` 를 통해 폴더 및 하위 폴더의 소유자 및 권한을 변경 가능하다.
```powershell
icacls.exe "C:\temp" /setowner Administrators /q /c /t
```
* [icacls Microsoft](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)

## 2. Windows Service 권한
특정 서비스의 실행 권한을 알고싶으면 아래 명령어를 통해 확인 가능하다.

`sc.exe sdshow "Service Name"`

해당 명령어를 통해 보게되면 SDDL이 출력된다. SDDL(Security Descriptor Definition Language)는 보안 설명자 정의 언어로 아래와 같은 형태로 구성되어있다.
`D:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCLCSWLOCRRC;;;AU)(A;;CCLCSWRPWPDTLOCRRC;;;PU)`

D:는 DACL(Discretionary Access Control List)을 의미한다. 괄호 () 안의 각 항목은 ACE(Access Control Entry)를 나타낸다. 각각의 ACE는 특정 사용자 또는 그룹에 대한 권한을 정의한다.
* BA = Built-in Administartors
* SY = Local System
* AU = Authenticated Users
