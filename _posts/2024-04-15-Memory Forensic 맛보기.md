---
title: Memory Forensic 맛보기
author: heogi
date: 2024-04-15
categories:
  - Forensic
  - Memory
tags:
  - Forensic
  - Memory
  - Volatility
comments: true
---
## Description
Defcon DFIR CTF 2019에 출제된 Memory Dump를 통해 수상한 프로세스에 대하여 분석해본다.

문제에서 제공되는 파일은 `Adam Ferrante - Triage-Memory.mem` 이고 `volatility` 를 통해 메모리 분석을 진행한다.
```powershell
volatility.exe -f Adam Ferrante - Triage-Memory.mem --profile=Win7SP1x64 pslist > pslist.txt
volatility.exe -f Adam Ferrante - Triage-Memory.mem --profile=Win7SP1x64 pstree > pstree.txt
volatility.exe -f Adam Ferrante - Triage-Memory.mem --profile=Win7SP1x64 netscan > netscan.txt
volatility.exe -f Adam Ferrante - Triage-Memory.mem --profile=Win7SP1x64 dlllist > dlllist.txt
```

추출이 완료되면 아래처럼 4개의 txt 파일이 존재한다.

![](../assets/img/Pasted%20image%2020240415224710.png)
## 악성 프로세스로 의심되는 파일

먼저 `dlllist.txt` 파일에서 실행된 `command line`을 모두 살펴본다.

```powershell
PS C:\Users\heogi-windows-vm\Downloads\DFIR_memory> type .\dlllist.txt | findstr "command line"
Command line : \SystemRoot\System32\smss.exe
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
Command line : wininit.exe
Command line : winlogon.exe
Command line : C:\Windows\system32\services.exe
Command line : C:\Windows\system32\lsass.exe
Command line : C:\Windows\system32\lsm.exe
Command line : C:\Windows\system32\svchost.exe -k DcomLaunch
Command line : C:\Windows\system32\svchost.exe -k RPCSS
Command line : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
Command line : C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
Command line : C:\Windows\system32\svchost.exe -k netsvcs
Command line : C:\Windows\system32\svchost.exe -k LocalService
Command line : C:\Windows\system32\svchost.exe -k NetworkService
Command line : C:\Windows\System32\spoolsv.exe
Command line : C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork
Command line : "C:\Program Files\Common Files\Microsoft Shared\ClickToRun\OfficeClickToRun.exe" /service
Command line : "taskhost.exe"
Command line : taskeng.exe {E9F45B18-C0E3-42EE-9422-2FAB865F5A64}
Command line : "C:\Windows\system32\Dwm.exe"
Command line : C:\Windows\Explorer.EXE
Command line : "C:\Program Files (x86)\FileZilla Server\FileZilla Server.exe"
Command line : "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
Command line : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
Command line : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
Command line : "C:\Program Files\VMware\VMware Tools\VMware CAF\pme\bin\ManagementAgentHost.exe"
Command line : "C:\Program Files (x86)\FileZilla Server\FileZilla Server Interface.exe"
Command line : C:\Windows\system32\dllhost.exe /Processid:{02D4B3F1-FD88-11D1-960D-00805FC79235}
Command line : C:\Windows\System32\msdtc.exe
Command line : C:\Windows\system32\wbem\wmiprvse.exe
Command line : C:\Windows\system32\SearchIndexer.exe /Embedding
Command line : "C:\Program Files\Windows Media Player\wmpnetwk.exe"
Command line : C:\Windows\system32\svchost.exe -k LocalServiceAndNoImpersonation
Command line : "C:\Windows\system32\notepad.exe"
Command line : C:\Windows\system32\wbem\wmiprvse.exe
Command line : "C:\Program Files (x86)\Microsoft Office\root\Office16\EXCEL.EXE"
Command line : "C:\Windows\System32\cmd.exe"
Command line : \??\C:\Windows\system32\conhost.exe "-3665641233614004741886043129-1340869214-8780857402033447618-1868066120-1609534987"
Command line : taskeng.exe {7F35D633-1462-41D2-B75C-0C83793D50AF}
Command line : C:\Windows\system32\sppsvc.exe
Command line : C:\Windows\System32\svchost.exe -k secsvcs
Command line : "C:\Program Files (x86)\Microsoft Office\root\Office16\OUTLOOK.EXE"
Command line : "C:\Windows\system32\taskmgr.exe" /4
Command line : "C:\Windows\system32\StikyNot.exe"
Command line : "C:\Windows\system32\calc.exe"
Command line : "C:\Program Files (x86)\Internet Explorer\iexplore.exe" -Embedding
Command line : "C:\Program Files (x86)\Internet Explorer\iexplore.exe" SCODEF:3576 CREDAT:79873
Command line : "C:\Users\Bob\Desktop\hfs.exe"
Command line : "C:\Program Files (x86)\Microsoft Office\root\Office16\POWERPNT.EXE"
Command line : "C:\Program Files\AccessData\FTK Imager\FTK Imager.exe"
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=watcher --main-thread-id=3272 --on-initialized-event-handle=308 --parent-handle=312 /prefetch:6
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=utility --field-trial-handle=924,2132560186875629139,18153288653831290455,131072 --lang=en-US --service-sandbox-type=network --service-request-channel-token=3848528183961585196 --mojo-platform-channel-handle=1320 /prefetch:8
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=renderer --field-trial-handle=924,2132560186875629139,18153288653831290455,131072 --service-pipe-token=5128242365178584495 --lang=en-US --instant-process --enable-offline-auto-reload --enable-offline-auto-reload-visible-only --device-scale-factor=1 --num-raster-threads=1 --service-request-channel-token=5128242365178584495 --renderer-client-id=7 --no-v8-untrusted-code-mitigations --mojo-platform-channel-handle=2268 /prefetch:1
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=renderer --field-trial-handle=924,2132560186875629139,18153288653831290455,131072 --service-pipe-token=16174022435611898554 --lang=en-US --enable-offline-auto-reload --enable-offline-auto-reload-visible-only --device-scale-factor=1 --num-raster-threads=1 --service-request-channel-token=16174022435611898554 --renderer-client-id=8 --no-v8-untrusted-code-mitigations --mojo-platform-channel-handle=2528 /prefetch:1
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=renderer --field-trial-handle=924,2132560186875629139,18153288653831290455,131072 --service-pipe-token=6187966837059053897 --lang=en-US --extension-process --enable-offline-auto-reload --enable-offline-auto-reload-visible-only --device-scale-factor=1 --num-raster-threads=1 --service-request-channel-token=6187966837059053897 --renderer-client-id=4 --no-v8-untrusted-code-mitigations --mojo-platform-channel-handle=2536 /prefetch:1
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=renderer --field-trial-handle=924,2132560186875629139,18153288653831290455,131072 --disable-gpu-compositing --service-pipe-token=2094820586012107996 --lang=en-US --enable-offline-auto-reload --enable-offline-auto-reload-visible-only --device-scale-factor=1 --num-raster-threads=1 --service-request-channel-token=2094820586012107996 --renderer-client-id=13 --no-v8-untrusted-code-mitigations --mojo-platform-channel-handle=3844 /prefetch:1
Command line : "C:\Windows\System32\wscript.exe" //B //NOLOGO %TEMP%\vhjReUDEuumrX.vbs
Command line : "C:\Users\Bob\AppData\Local\Temp\rad93398.tmp\UWkpjFjDzM.exe"
Command line : C:\Windows\system32\cmd.exe
Command line : \??\C:\Windows\system32\conhost.exe "-2105419398-1914772985-1870609020-898984210-362940684623073995-430731961-49370805"
```

추출된 값 중 인자 값 없이 실행된 `svchost.exe` 도 확인해본다.

`svchost.exe` 는 모두 `-k` 인자 값과 함께 실행되어 정상적인 프로세스라고 추측할 수 있다.

```powershell
PS C:\Users\heogi-windows-vm\Downloads\DFIR_memory> type .\dlllist.txt | findstr "command line" | findstr svchost
Command line : C:\Windows\system32\svchost.exe -k DcomLaunch
Command line : C:\Windows\system32\svchost.exe -k RPCSS
Command line : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
Command line : C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
Command line : C:\Windows\system32\svchost.exe -k netsvcs
Command line : C:\Windows\system32\svchost.exe -k LocalService
Command line : C:\Windows\system32\svchost.exe -k NetworkService
Command line : C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork
Command line : C:\Windows\system32\svchost.exe -k LocalServiceAndNoImpersonation
Command line : C:\Windows\System32\svchost.exe -k secsvcs
```

또한 `svchost.exe`를 실행한 부모 프로세스를 확인해보면 
모두 `PPID 476` 번으로 `services.exe` 프로세스에 의해 정상적으로 실행된 것을 확인할 수 있다.

```powershell
PS C:\Users\heogi-windows-vm\Downloads\DFIR_memory> type .\pstree.txt | findstr services.exe
. 0xfffffa8005680910:services.exe                     476    380     12    224 2019-03-22 05:31:59 UTC+0000

PS C:\Users\heogi-windows-vm\Downloads\DFIR_memory> type .\pstree.txt | findstr svchost.exe
.. 0xfffffa800583db30:svchost.exe                    1028    476     19    307 2019-03-22 05:32:05 UTC+0000
.. 0xfffffa8005775b30:svchost.exe                     796    476     15    368 2019-03-22 05:32:03 UTC+0000
.. 0xfffffa80057beb30:svchost.exe                     932    476     10    568 2019-03-22 05:32:03 UTC+0000
.. 0xfffffa800432f060:svchost.exe                    3300    476     13    346 2019-03-22 05:34:15 UTC+0000
.. 0xfffffa800577db30:svchost.exe                     820    476     33   1073 2019-03-22 05:32:03 UTC+0000
.. 0xfffffa800570d060:svchost.exe                     672    476      7    341 2019-03-22 05:32:02 UTC+0000
.. 0xfffffa8005c4ab30:svchost.exe                    2888    476     11    152 2019-03-22 05:32:20 UTC+0000
.. 0xfffffa80056e1060:svchost.exe                     592    476      9    375 2019-03-22 05:32:01 UTC+0000
.. 0xfffffa80057e4560:svchost.exe                     232    476     15    410 2019-03-22 05:32:03 UTC+0000
.. 0xfffffa800575e5b0:svchost.exe                     764    476     20    447 2019-03-22 05:32:02 UTC+0000
```

다음으로 `pslist.txt` 파일을 살펴보면 `UWkpjFjDzM.exe` 라는 수상한 이름을 가진 프로세스를 확인할 수 있다.

```
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8005a1d9e0 UWkpjFjDzM.exe         3496   5116      5      109      1      1 2019-03-22 05:35:33 UTC+0000                                 
0xfffffa8005bb0060 cmd.exe                4660   3496      1       33      1      1 2019-03-22 05:35:36 UTC+0000                                 
0xfffffa8005c1ab30 conhost.exe            4656    372      2       49      1      0 2019-03-22 05:35:36 UTC+0000       
```

`pstree.txt` 에서 해당 프로세스의 실행 구조 및 리니지를 살펴보면 아래와 같다.

`explorer.exe` > `hfs.exe` > `wscript.exe` > `UWkpjFjDzM.exe` > `cmd.exe`

```
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xfffffa8003de39c0:explorer.exe                     1432   1308     28    976 2019-03-22 05:32:07 UTC+0000
. 0xfffffa80042aa430:cmd.exe                         1408   1432      1     23 2019-03-22 05:34:12 UTC+0000
. 0xfffffa8005d067d0:StikyNot.exe                    1628   1432      8    183 2019-03-22 05:34:42 UTC+0000
. 0xfffffa800474c060:OUTLOOK.EXE                     3688   1432     30   2023 2019-03-22 05:34:37 UTC+0000
. 0xfffffa8004798320:calc.exe                        3548   1432      3     77 2019-03-22 05:34:43 UTC+0000
. 0xfffffa80053d3060:POWERPNT.EXE                    4048   1432     23    765 2019-03-22 05:35:09 UTC+0000
. 0xfffffa8004905620:hfs.exe                         3952   1432      6    214 2019-03-22 05:34:51 UTC+0000
.. 0xfffffa8005a80060:wscript.exe                    5116   3952      8    312 2019-03-22 05:35:32 UTC+0000
... 0xfffffa8005a1d9e0:UWkpjFjDzM.exe                3496   5116      5    109 2019-03-22 05:35:33 UTC+0000
.... 0xfffffa8005bb0060:cmd.exe                      4660   3496      1     33 2019-03-22 05:35:36 UTC+0000
```

