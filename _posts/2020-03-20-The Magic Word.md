---
title: Angstromctf2020 - The Magic Word
author: heogi
categories: [CTF-Writeups]
tags: [web CTF-Writeups Angstromctf2020]
---
# The Magic Word

![error](https://raw.githubusercontent.com/heogi/heogi.github.io/master/_posts/CTF%20Writeups/angstromctf2020/The%20Magic%20Word/2020-03-20-13-46-34.png)

![error](https://raw.githubusercontent.com/heogi/heogi.github.io/master/_posts/CTF%20Writeups/angstromctf2020/The%20Magic%20Word/2020-03-20-13-50-31.png)

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
