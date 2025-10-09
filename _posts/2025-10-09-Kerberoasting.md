---
title: Kerberoasting
author: heogi
date: 2025-10-09
categories:
  - Kerberos
  - Kerberoasting
  - Active Directory
tags:
  - Kerberos
  - Kerberoasting
  - Active-Directory
comments: true
---
## Kerberoasting
Kerberoasting은 Kerberos 프로토콜의 정상적인 동작(서비스 티켓 발급)을 악용해서 서비스 계정의 암호화된 패스워드를 확보하고 이를 오프라인에서 크랙하여 계정 크리덴셜을 탈취하는 기법이다.

## Kerberos란 
Kerberos란 안전하지 않은 네트워크에서 사용자나 컴퓨터가 서로의 신원을 안전하게 인증하기 위해 사용하는 티켓 기반의 네트워크 인증 프로토콜이다.
![|1288x1176](../assets/img/2025-10-09-Kerberoasting-1760010354948.png)