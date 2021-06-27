---
title: "MongoDB ObjectID"
header:
  teaser: ""
categories:
  - Web
tags:
  - MongoDB
  - Web
toc: true
---

# MongoDB - ObjectID 구조

MongoDB의 ObjectID는 저장된 개체를 식별하기 위한 고유 값이다.

해당 값은 TimeStamp(4Byte), Random(5Byte), Incrementing Counter(3Byte)로 구성되어있다.

아래는 ObjectID의 예시 값이다.

```
Timestamp   Random    Inc
60d80a61  52214b8321 7b74e0
60d80a64  52214b8321 7b74e1
60d80a67  52214b8321 7b74e2
60d80a6b  52214b8321 7b74e3
```


