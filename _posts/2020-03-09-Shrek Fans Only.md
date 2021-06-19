---
title: UTCTF2020 - Shrek Fans Only
author: heogi
categories: [CTF-Writeups]
tags: [web CTF-Writeups]
---
# Shrek-Fans-Only  

> ## Overview  

![이미지001](https://user-images.githubusercontent.com/45466073/122631579-8ddd5100-d107-11eb-879c-9c534c835c5e.png)

링크로 들어가면 아래와 같은 페이지가 나온다.  

![이미지002](https://user-images.githubusercontent.com/45466073/122631585-92a20500-d107-11eb-9c36-5e913c9b4227.png)
![이미지004](https://user-images.githubusercontent.com/45466073/122631592-99307c80-d107-11eb-8a88-7a9c9415423c.png)

슈렉 이미지는 getimg.php 페이지에서 base64형태로 인코딩되어 이미지 파일의 이름이 입력되어있다.  
이를 이용해 LFI가 가능하여 passwd 파일을 읽을 수 있었다.

![이미지005](https://user-images.githubusercontent.com/45466073/122631594-9d5c9a00-d107-11eb-957c-e9a89343b609.png)


이후에 FLAG가 있을만한 여러파일을 뒤적거리다 포기......  
그 와중에 .git 폴더를 발견하여 여러가지 뒤적거려 봤지만  
git 구조에 대한 지식이 없고 해볼 생각을 안하고 그냥 넘어갔다 ㅡㅡ  
답을 보고 풀었다.  commit을 보고 blob?? 을 찾아서 요청을 보내면 된다고한다.

![이미지007](https://user-images.githubusercontent.com/45466073/122631596-a0578a80-d107-11eb-9835-14816d0a1d20.png)

![이미지006](https://user-images.githubusercontent.com/45466073/122631601-a51c3e80-d107-11eb-882c-636783db02d4.png)


