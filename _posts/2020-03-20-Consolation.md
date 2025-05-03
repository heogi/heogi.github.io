---
title: Angstromctf2020 - Consolation
header:
  teaser: ""
categories:
  - CTF
  - Angstromctf2020
tags:
  - Angstromctf2020
  - CTF
toc: true
---

# Consolation
---
![](../assets/img/Pasted%20image%2020240330202946.png)

문제에 접속하면 버튼이 보인다.
![](../assets/img/Pasted%20image%2020240330202953.png)


```html
<html>
<head>
	<title>consolation</title>
</head>

<body style="padding: 20px">

$<span id="monet">0</span>

<br /><br /><br />

<button onclick="nofret()" style="height:150px; width:150px;">pay me some money</button>

<script src="iftenmillionfireflies.js"></script>

</body>
</html>
```

소스에는 js파일이 하나있고 버튼을 클릭하면 nofret()이라는 함수가 실행된다.

js파일을 살펴보면 nofret()함수가 존재한다.

```javascript
function nofret() {
    document[_0x4229('0x95', 'kY1#')](_0x4229('0x9', 'kY1#'))[_0x4229('0x32', 'yblQ')] = parseInt(document[_0x4229('0x5e', 'xtR2')](_0x4229('0x2d', 'uCq1'))['innerHTML']) + 0x19;
    console[_0x4229('0x14', '70CK')](_0x4229('0x38', 'rwU*'));
    console['clear']();
}
```

함수를 실행한뒤에 console 을 clear 시킨다. 해당 내용을 지우고 다시 nofret()함수를 실행시키면 
![](../assets/img/Pasted%20image%2020240330203001.png)

flag가 나타난다.
```text
actf{you_would_n0t_beli3ve_your_eyes}
```
