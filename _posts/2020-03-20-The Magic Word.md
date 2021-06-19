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

![2020-03-20-13-46-34](https://user-images.githubusercontent.com/45466073/122631526-363ee580-d107-11eb-81e6-5fc428dbb784.png)

![2020-03-20-13-50-31](https://user-images.githubusercontent.com/45466073/122631528-3808a900-d107-11eb-8bbf-34e9ff6458db.png)

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
