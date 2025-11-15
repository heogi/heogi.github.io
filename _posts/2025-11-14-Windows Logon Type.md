---
title: Windows Logon Type
author: heogi
date: 2025-11-14
categories:
  - Incident Response
tags:
  - Incident_Response
  - windows
comments: true
---
## Windows Logon Type
EDR 또는 Event VIewer를 분석하다보면 여러 Windows Logon Type이 보인다.   
주로 발생하는 Logon Type에 대해 정리하고 어떤 상황에서 해당 Logon Type이 발생하는지 정리해본다.

| Logon Type | Logon Title       | Description                                                                                                                                               | Example                                                 |
| ---------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| 2          | Interactive       | 사용자가 키보드/모니터를 통해 직접 콘솔에서 로그인 하는 경우 발생하는 로그온 타입                                                                                                            | PC에 직접 접근해서 로그인                                         |
| 3          | Network           | 네트워크를 통한 자원 접근 시 발생하는 로그온 타입.<br>실제 사용자 세션이나 화면 로그온 없이도 인증 토큰만 생성되며,<br>이 토큰을 통해 SMB 같은 네트워크 자원에 정상적으로 접근할 수 있다.<br>또한 RDP와 같은 원격 데스크탑 연결 시 인증 과정에서 발생한다. | SMB, PSExec, WMI 원격 쿼리, Powershell WinRM, RDP 인증 진행 시 등 |
| 4          | Batch             | 스케줄된 작업(Job, Scheduled Task)이 실행될 때 자동으로 사용되는 로그온 타입                                                                                                      | Task Scheduler에서 등록된 작업 실행                              |
| 5          | Service           | Windows 서비스가 특정 계정으로 실행될 때 발생하는 서비스 계정 로그온                                                                                                                | SQL Server 서비스, IIS 등 기타 Windows Service가 서비스 계정 사용     |
| 10         | RemoteInteractive | RDP와 같은 원격 GUI 세션 생성 완료 시 발생                                                                                                                              | 원격 데스크탑(mstsc.exe)                                      |

## Logon Type 별 정상 프로세스 트리 패턴

```text
# Logon Type 2 (Interactive)
winlogon.exe  
 └─ userinit.exe  
     └─ explorer.exe  
         └─ 사용자가 실행한 프로세스(cmd.exe, powershell.exe 등)
         
# Logon Type 4 (Batch)
taskeng.exe  
 └─ cmd.exe / powershell.exe / python.exe (Task Scheduled 작업 내용)
 
# Logon Type 5 (Services)
services.exe  
 └─ <service>.exe
 
# Logon Type 10 (RemoteInteractive)
winlogon.exe  
 └─ userinit.exe  
     └─ explorer.exe  
         └─ 사용자가 실행한 프로세스
```


> 참고
>  - https://learn.microsoft.com/ko-kr/windows-server/security/windows-authentication/windows-logon-scenarios
> {: .prompt-info }

