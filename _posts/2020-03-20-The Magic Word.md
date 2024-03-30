---
title: "Angstromctf2020 - The Magic Word"
header:
  teaser: ""
categories:
  - CTF
tags:
  - Angstromctf2020
  - CTF
toc: true
---


# The Magic Word

![](../assets/img/Pasted%20image%2020240330203425.png)

![](../assets/img/Pasted%20image%2020240330203430.png)

페이지 소스코드에 동작하는 스크립트가 있다.

```javascript
<script>
    var msg = document.getElementById("magic");
    setInterval(function() {
        if (magic.innerText == "please give flag") {
            fetch("/flag?msg=" + encodeURIComponent(msg.innerText))
                .then(res => res.text())
                .then(txt => magic.innerText = txt.split``.map(v => String.fromCharCode(v.charCodeAt(0) ^ 0xf)).join``);
        }
    }, 1000);
</script>
```

요구사항에 따라 magic의 innerText에 ```please give flag```를 넣으면 된다.

```text
actf{1nsp3c7_3l3m3nt_is_y0ur_b3st_fri3nd}
```
