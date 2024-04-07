---
title: MITRE ATT&CK
author: heogi
date: 2023-08-20
categories:
  - Incident Response
  - MITRE ATT&CK
tags:
  - MITRE_ATTACK
  - Incident_Response
  - MITRE_FRAMEWORK
comments: true
---
## **1\. MITRE ATT&CK**

-   공개된 침해사고 분석 보고서, 악성코드 분석 보고서, 위협 그룹에 대한 정보를 분석하여 공격자들의 TTPs를 분석하여 체계적으로 집대성한 저장소  
## **2\. TTPs(Tatics Techniques Procedures)**

-   Tatics : 위협 행위의 목적
-   Techniques : 위협 행위의 목적을 달성하기 위한 기술
-   Procedures : 테크닉을 구현하기 위한 구체적인 절차와 방법  
   
## **3\. MITRE ATT&CK Matrix**

-   테크닉/서브테크닉을 전술을 기준으로 분류해 놓은 표
-   Matrix 종류
	- Enterprise : 윈도우, MacOS, Linux, Cloud, Office 등에 관련된 전술 및 테크닉을 수록한 버전
    - Mobile : Android 및 IOS 관련된 전술 및 테크닉을 수록한 버전
    - ICS : 산업제어시스템과 관련된 전술 및 테크닉을 수록한 버전

[##_Image|kage@NIrlj/btsEXqZqCFT/nfptq5M3XZAXPow7nGkzaK/img.png|CDM|1.3|{"originWidth":2950,"originHeight":950,"style":"alignCenter"}_##]

## **4.  Atomic Red Team**

-   TTPs 기반 단위 테스트 도구 모음
-   [https://github.com/redcanaryco/atomic-red-team](https://github.com/redcanaryco/atomic-red-team)

 [GitHub - redcanaryco/atomic-red-team: Small and highly portable detection tests based on MITRE's ATT&CK.

Small and highly portable detection tests based on MITRE's ATT&CK. - redcanaryco/atomic-red-team

github.com](https://github.com/redcanaryco/atomic-red-team)

## **5.  Top ATT&CK Techniques**

-   빈번하게 발생하는 Techniques
-   MITRE (현재는 Ransomeware에 대해서만 분석되어있음)
    -   [https://top-attack-techniques.mitre-engenuity.org/](https://top-attack-techniques.mitre-engenuity.org/)
-   Red Canary
    -   [https://redcanary.com/threat-detection-report/techniques/](https://redcanary.com/threat-detection-report/techniques/)
-   Picus
    -   [https://www.picussecurity.com/resource/the-top-ten-mitre-attck-techniques](https://www.picussecurity.com/resource/the-top-ten-mitre-attck-techniques)