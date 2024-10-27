---
title: cURL-Based Call
author: heogi
date: 2024-10-27
categories:
  - CTF
  - Dreamhack
tags:
  - web
comments: true
---
## 1. Description

curl 명령어를 기반으로 백엔드 서비스의 API를 호출하는 웹 서비스입니다.  
백엔드 서비스의 GET /admin 엔드포인트를 통해 플래그를 얻을 수 있습니다.
서비스의 취약점을 찾고 익스플로잇하여 플래그를 획득하세요!
플래그 형식은 DH{...} 입니다.

## 2. Analysis
curl 기반으로 API 기능을 구현한 웹 페이지이다.
웹앱을 통해 Backend 서버에 /admin 을 호출하여 Flag를 획득할 수 있다.

![](../assets/img/Pasted%20image%2020241028082030.png)

호출할 수 있는 기능은 아래 4가지이다.
* create
* read
* update
* delete

각각 아래 subprocess로 생성된 curl 명령을 통해 Backend에 API를 호출한다.

```python
def curl_backend_api(path, method, client_host, token=None, data=None):
	try:
		args = [
			'/usr/bin/curl', f'{BACKEND_BASE_URL}{path}',
			'-H', f'Simple-Token: {token}',
			'-H', f'X-Forwarded-For: {client_host}',
			'-H', 'Content-Type: application/json',
			'-X', method,
			'--ignore-content-length',
			'--max-time', '0.3',
			'-d', json.dumps(data) if data else ''
		]
		res = subprocess.run(args, capture_output=True)
		return res.stdout.decode()
	except:
		return None
```

각 요청에는 `/auth` API를 통해 발급되는`token` 값을 포함하여 요청한 클라이언트를 구분한다.



## 3. Attack
