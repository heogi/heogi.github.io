---
title: UTCTF2020 - Shrek Fans Only
header:
  teaser: ""
categories:
  - CTF
  - UTCTF2020
tags:
  - UTCTF2020
  - CTF
toc: true
---
# Shrek-Fans-Only  

> ## Overview  

![](../assets/img/Pasted%20image%2020240330202603.png))

링크로 들어가면 아래와 같은 페이지가 나온다.  

![](../assets/img/Pasted%20image%2020240330202721.png)
![](../assets/img/Pasted%20image%2020240330202728.png)이를 이용해 LFI가 가능하여 passwd 파일을 읽을 수 있었다.
![](../assets/img/Pasted%20image%2020240330202745.png)


이후에 FLAG가 있을만한 여러파일을 뒤적거리다 포기......  
그 와중에 .git 폴더를 발견하여 여러가지 뒤적거려 봤지만  
git 구조에 대한 지식이 없고 해볼 생각을 안하고 그냥 넘어갔다  
답을 보고 풀었다.  commit을 보고 blob?? 을 찾아서 요청을 보내면 된다고한다.

![](../assets/img/Pasted%20image%2020240330202755.png)
![](../assets/img/Pasted%20image%2020240330202811.png)

