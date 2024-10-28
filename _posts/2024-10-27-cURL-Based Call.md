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
* `create`
* `read`
* `update`
* `delete`

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
이는 위 curl 요청을 보내는 `curl_backend_api`에서 `Simple-Token: {token}`으로 구현되어있다.

아래는 `create` 함수의 구현이다.
`-d` 옵션을 통해 curl에 삽입되는 data를 입력받으며, `request.remote_addr`을 통해 클라이언트의 IP를 식별한다.

```python
@app.route('/create', methods=['POST'])
def post_create():
	title = request.form.get('title')
	content = request.form.get('content')
	author = request.form.get('author')
	
	if not isinstance(title, str) \
			or not isinstance(content, str) \
			or not isinstance(author, str):
		abort(400)

	data = {'title': title, 'content': content, 'author': author}
	res = curl_backend_api('/posts', POST, request.remote_addr, g.simple_token, data)
	
	if res is None:
		abort(400)

	return render_template('/api_result.html', simple_token=g.simple_token, res=beautify(res))
```

아래는 `/admin` 함수의 구현이다.
`X-Forwarded-For` Header를 통해 클라이언트 IP를 식별하여, IP가 `127.0.0.1`이 아닐경우 호출이 불가하도록 구현되어있다.

```python
@app.get('/admin')
async def get_admin(request: Request):
	x_forwarded_for = request.headers.get('X-Forwarded-For')
	if x_forwarded_for != '127.0.0.1':
		return JSONResponse(status_code=401, content=None)
	return {'message': FLAG}
```

## 3. Attack
`curl_backend_api` 함수의 `token` 변수는 사용자가 조작이 가능하며, 입력 값에 대한 제어가 없어 CRLF Injection 공격에 취약하다.
이를 통해 `X-Forwarded-For` 헤더를 추가로 삽입하는것이 가능하며, 새로운 `HTTP Request Line` 을 추가하여 요청하는것도 가능하다.


