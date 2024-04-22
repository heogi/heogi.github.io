---
title: AWS Systems Manager
author: heogi
date: 2023-11-18
categories:
  - Cloud
  - Systems Manager
tags:
  - Cloud
  - AWS
  - Systems-Manager
comments: true
---
## **AWS Systems Manager**

![](../assets/img/Pasted%20image%2020240422145209.png)
## **Fleet Manager**

-   EC2 Instance에 SSM Agent가 설치되어있다면, 해당 에이전트를 통해 명령어 전달 및 수행, 성능 확인, 브라우저를 통한 원격 터미널 접속 가능
-   단일 대상, Tag, Resource Group 등을 통해 조건에 맞는 대상들만 관리 가능하다.
-   Document를 통해 json, yaml 형식으로 명령어 수행이 가능하다.
-   SSM 관련 적절한 IAM Role 설정이 필요하다.

## **Automations**

![](../assets/img/Pasted%20image%2020240422145215.png)

-   보안 패치 등 자동화
-   Fleet Manager의 Document와 동일하게 RunBooks 라는게 있다.

## **Paramter Store**

![](../assets/img/Pasted%20image%2020240422145232.png)

-   Application 에서 사용되는 파라미터에 대한 보관, 버전 관리 기능, 계층 구조로 관리 가능하다.
-   EventBridge를 통해 알람을 받을 수 있으며, Cloud Formation과 통합되어있다.  

![](../assets/img/Pasted%20image%2020240422145238.png)

## **Patch Manager**

-   EC2, On-Promises의 운영체제, 어플리케이션, Security Update의 자동화된 패치를 지원하는 기능
-   Windows, Linux, MacOS를 지원
-   운영체제의 Major 버전 업데이트는 미지원 (ex. Windows Server 2016 -> Windows Server 2019)
-   Patch 규정 준수 로그를 S3에 저장가능

## **Session Manager**

-   EC2, OnPromises 서버에 Secure Shell 연결을 해주는 기능
-   SSH, Bastion Host, SSH Key 없이 접속이 가능하게 해준다.  
    EC2 Instance의 Security Group에 SSH 접속이 Deny 되어있어도 Session Manager를 통해 Secure Shell 접속 가능
-   AWS Console, SDK, CLI를 통해 사용 가능하며, Windows, Linux, MacOS를 지원한다.