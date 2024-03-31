---
title: Secure Mail
author: heogi
date: 2024-03-31
categories:
  - CTF
  - Dreamhack
tags:
  - CTF
comments: true
---
## **1\. Description**

중요한 정보가 적혀있는 보안 메일을 발견하였습니다.  
보안 메일의 비밀번호는 생년월일 6자리인 것으로 파악되나, 저희는 비밀번호 정보를 가지고 있지 않습니다.  
비밀번호를 알아내고 보안 메일을 읽어 중요한 정보를 알아내주세요!

## **2\. Analysis & Attack**

정답이 생년월일 범위로 한정되어 Bruteforce 진행한다.

개발자 도구에서 실행하면 data:image/png;base64 로 시작하는 문자열이 나오는데 브라우저 주소창에 입력하면 플래그의 사진이 나온다.

```javascript
#!ex.js
var yy = "90";
var mm = "01";
var dd = "01";

while(true){
	dd = String(Number(dd) + 1);
	if(dd=="32"){
		dd="01";
		mm = String(Number(mm) + 1);
	}
	if(mm=="13"){
		mm="01";
		yy = String(Number(yy) + 1);
		console.log(yy);
	}
	if(yy=="100"){
		yy="01";
	}
	if(dd.length == 1) dd = "0" + dd;
	if(mm.length == 1) mm = "0" + mm;
	
	document.getElementById("pass").value=yy+mm+dd;
    document.getElementById("bt").click();
 }
```