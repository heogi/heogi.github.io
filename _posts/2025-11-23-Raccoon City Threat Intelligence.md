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
## Summary
전 직원에게 CEO 명의로 복지 포털 로그인 요청 이메일이 발송되었습니다.   
CEO는 발송 사실을 부인했습니다. TI 팀은 원본 이메일 `client_request.eml`을 확보했습니다.
의심 이메일의 발송 경로와 인프라를 추적하라.

## Client_request.eml 분석
`client_request.eml` 확인 시 `ceo_forward.eml` 파일이 확인된다.
해당 메일에는 CEO로 부터 FW 된 피싱 메일인 `original_mass_email.eml` 첨부되어있다.
![|394x243](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763905504005.png)
피싱 메일에 첨부된 링크는 `http://vpn.racooncoin.site/` (Typosquatting) 이며 Raccoon Coin 내부 임직원들이 사용하는 VPN Web Portal로 위장한것으로보인다.
![|590x369](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763905591944.png)

## Phising Site 분석
Phising Site 의 코드를 확인해보면 아래와 같은 수상한 Javascript가 확인된다.
해당 사이트에 사용자가 입력한 사내 이메일, 패스워드 등 을 `140.238.194.224` 서버로 전송하고 3초 뒤 원본 사이트인 `www.raccooncoin.site`로 이동시킨다.
해당 Phising Site를 통해 공격자는 Raccooncoin 임직원의 사내 계정 및 패스워드를 확보할 수 있었다.
```javascript
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
```

## Credential Harvest 서버 분석
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

이 중 임직원 정보를 수집하기 위해 오픈되어있는 80 포트 외 8081 SimpleHTTP 서비스에 접속해보면 `raccooncoin_info.zip` 파일이 확인된다.

![218x97](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1763996967110.png)

압축을 해제하면 아래의 파일들이 확인된다.
* `db.sql` - Raccooncoin 대표 및 임직원의 ID, Email, Password Hash가 기록된 SQL 파일
* `plan.txt` - 공격 계획에 대한 상세한 설명이 기재된 파일
* `Source Code.zip` - Spark Repo의 소스코드

`plan.txt` 파일에는 Raccooncoin 공격을 위한 계획이 수립 되어있는것을 확인할 수 있다.

```text
[0x01] Initial Recon
--------------------
- Target: raccooncoin.site (KR-based crypto exchange)
- Goal: Obtain internal access to Staff VPN / admin panel
- Approach: OSINT → impersonation → phishing → VPN creds → pivot
중략...

[0x02] Social Engineering Plan
Idea: Create fake LinkedIn + GitHub profiles to impersonate internal staff or trusted vendors (for dropper purpose)

1) Fake LinkedIn profile #1 - their tech lead
   - Name: "Soyeong Park"
   - Title: "Teach Lead at Raccoon Coin"
   - Pitch: for internal vpn training or just try out fake logon attempt to internal employeee
   - Connect with:
     - Tony Raccoon
     - anyone listing "RaccoonCoin" or "RaccoonCoin Exchange" as employer.
중략...

1) Fake GitHub profile - TBU more
   - Use the tech lead identity (with email verification with racooncoin.site )
   - Bio: "Tech lead raccooncoin."
   - Mirror some public code (fork from open-source projects).
   - Upload fake "internal" tools repo later:
     - /scripts/vpn-helper.sh
     - /tools/raccoon-monitor.py
   - Idea: if devs Google random errors, they might land on this GitHub and trust it as internal.
   - Could embed malicious curl|bash installer in README later.
중략...

[0x03] Phishing Scenario
------------------------
Objective: Get staff to log into attacker-controlled VPN portal, extract DB.
중략...

[0x05] Future Steps / To-Do
---------------------------
[ ] Finish fake LinkedIn + GitHub profiles and age them for a few days.
[ ] Join Korean security / crypto groups and casually interact to build trust.
[ ] Finalize phishing email template in both English and Korean.
[ ] Deploy cloned VPN login page and test credential logging.
[ ] Deploy cloned VPN login page and test credential logging.
[ ] Prepare OSINT trail so that investigators (CTF players) can:
    - find this notes file
    - pivot from leaked SQL → email addresses → social media → fake profiles → onion/redirector infra.
```

## Fake Linkedin Profile
`plan.txt` 파일에 기반하여 공격자들이 사용했을 것으로 추정되는 Soyeong Park 계정의 Fake Linkedin Profile을 추적한다.
`www.raccooncoin.site` 페이지에 접속하면 Raccooncoin 임직원의 간단한 소개가 기재 되어 있는것을 확인할 수 있다.
![717x185](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764170222790.png)

Linkedin에서 `Soyeong Park raccoon coin` 으로 검색하면 Soyeong Park 계정의 프로필이 확인된다.
![|395x57](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764170930833.png)

## Fake Github Profile
github에 raccooncoin으로 서칭 시 raccooncoin-dev 레포가 확인된다.
![|675x387](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764385885855.png)

해당 레포를 clone 하여 `git show`명령어를 통해 commit 시간을 확인 할 수 있다.
commit 중 하나를 확인해보면 `Date: Wed Nov 12 21:45:25 2025 -0500`로 확인되고
![|598x221](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764386900899.png)
이를 통해 공격자는 UTC - 5 인 국가에서 commit을 한것으로 추정할 수 있다.

## Credential Harvest 서버 Initial access
commit 중 `d64dd40661b4f35cc74be42cbfa72703622e7aa4` 를 확인해보면 특정 서버의 ssh key가 노출된것이 확인된다.
![|580x254](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764387173190.png)

또다른 commit인 `4c8b9215b593d20b4d78558592e29777ef4e9162`를 확인하면, 해당 키는 `140.238.194.224`에 접속하기 위한 SSH Key인것을 추측할 수 있다.
![|340x189](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764505850279.png)

확인된 ssh key 및 passphrase 를 기반으로 Credential Harvest 서버에 접속하여 추가 조사를 진행한다.
```shell
ssh -i id_ed25519 spark@140.238.194.224
```
![598x365](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764433494816.png)
접속 후 `.bashrc` 파일을 확인해보면 아래와 같은 onion URL이 확인되며, C2 서버에 대한 IP를 환경 변수로 정의해놓았다.
```shell
#.bashrc
중략...
export ONION=http://w7kea3mqv3pq4rhnpavv3ezgyhtkj2v443oidlqtuj7aa72wu5yh2nqd.onion/
#export ONIONKEY=6BPIMPLNL5ISY5O3LGVVRUE7BEZMLFZ5WITY2XPZ26P45YGMJZIQ << maybe not needed
export C2=158.180.x.x
unset HISTFILE
export HISTSIZE=0
export HISTFILESIZE=0
```

`netstat` 을 통해 IP를 찾아보면 SSH를 통해 현재 연결되어있는것을 알 수 있다.
```shell
spark@redirector:~$ netstat -ano | grep 158.180.
tcp        0      0 10.0.0.65:22            158.180.6.169:40302     ESTABLISHED keepalive (1339.67/0/0)
```

## Onion URL 조사
`.bashrc` 파일에서 확인된 onion URL에 접속하여 추가 조사를 진행한다.
Tor 브라우저에서 onion URL에 접속하면 RAASNet(랜섬웨어 생성 서비스)로 접속되며 `Navigation` 탭에서 `Dashboard` 로 접속하면 아래와 같은 Victim, Artifacts 등의 화면이 확인된다.
![|1057x282](../assets/img/2025-11-23-Raccoon%20City%20Threat%20Intelligence-1764492324203.png)

Victims Overview에는 CEO인 tony.raccoon의 Workstation과 HR 직무로 추정되는 Workstation, DB 서버로 추정되는 srv-db-01 Server가 감염된것으로 확인된다.   
`Quick Artifacts` 중 `ransom_loader_v2.exe` 파일을 다운로드 후 해당 파일을 조사해보면, 파일은 `ASCII text` 파일이고 아래와 같은 스트링이 출력된다.

```shell
┌──(root㉿kali)-[~kali/Downloads]
└─# file ransom_loader_v2.exe     
ransom_loader_v2.exe: ASCII text

┌──(root㉿kali)-[~kali/Downloads]
└─# cat ransom_loader_v2.exe    
Placeholder: Windows loader binary (text-safe). Just a text file for educational purpose, just chill dude.

┌──(root㉿kali)-[~kali/Downloads]
└─# sha256sum ransom_loader_v2.exe    
e4c1572b153b10ed540f415dc436a87c7b46f0965daaa3ac98df3072925013e8  ransom_loader_v2.exe
```

이후 공격자는 해당 랜섬웨어 파일을 첨부하여 이메일을 임직원 또는 파트너사에게 발송, 랜섬웨어 감염을 통해 랜섬 획득이 목적일 것으로 추측된다.
## 결론
### 공격자 인프라
```text
Phishing Site(AWS S3 Static Hosting)
Credential Harvest, SSH Pivot Server(140.238.194.224)
C2 Server(158.180.6.169)
RAASNet(hxxp://w7kea3mqv3pq4rhnpavv3ezgyhtkj2v443oidlqtuj7aa72wu5yh2nqd.onion)
```

### 공격 과정 및 TTP
```text
> Typosquatting (T1583.001 / Acquire Infrastructure: Domains)
> Phishing Email (T1566.002 / Spearphising Link)
> Credential Harvest (T1589.001 / Credential)
> DB Extract (T1005 / Data from Local System)
```

### IOC
```text
#Domain
hxxps://vpn.racooncoin.site // Fake Internal VPN Server
hxxps://github.com/racconcoin // Fake Github Repository
https://www.linkedin.com/in/soyeong-park-5046b7391 // Fake Linkedin Profile

#IP
140.238.194.224 // Credential Harvest Server
158.180.6.169 // C2 Server

#Ransomware
e4c1572b153b10ed540f415dc436a87c7b46f0965daaa3ac98df3072925013e8 (ransom_loader_v2.exe) // SHA256
```

## 대응 방안
* 피싱 이메일 임직원 보안 교육
* Typosquating URL 선점
* 수집된 IOC에 대한 차단(백신, IDS, IPS, Firewall 등)
