---
title: Web Server vs WAS(Web Application Server)
author: heogi
date: 2023-09-29
categories:
  - Web
tags:
  - web
  - was
  - web-server
comments: true
---
## **1\. Web Server ?**

Web Server는 **단순히 정적인 페이지를 서비스**하기 위한 서버이다.

이미지 파일, 단순 HTML 파일 같은 정적인 파일들은 Web Server를 통해서 요청을 처리한다.

예) Apache Server, Nginx, IIS 등

## **2\. WAS(Web Application Server)?**

**동적인 페이지를 제공**하기 위해 DB 조회나 서비스를 위한 로직을 수행하는 서버이다. 상대적으로 부하가 많은 작업들이 진행된다.

예) Tomcat, JBoss, Jeus 등

![](../assets/img/Pasted%20image%2020240413012328.png)

## **3\. Web Server와 WAS를 따로 쓰는 이유**

1.  서버 부하 방지 정적인 페이지의 요청은 Web Server에서 처리하도록 하여 WAS의 부하를 방지한다.
2.  물리적으로 분리하여 보안 강화 SSL에 대한 암복호화 처리에 Web Server를 사용 WAS의 외부로의 직접적인 요청은 차단함으로써 보안 강화
3.  Scaling, Avalibility 무중단으로 Scale-In, Out 등에 유리하며, 여러대의 WAS중 하나가 서비스가 중단되더라도 Web Server에서 중단된 WAS로의 전달을 못 하도록 설정하여 장애 처리에 유리하다.