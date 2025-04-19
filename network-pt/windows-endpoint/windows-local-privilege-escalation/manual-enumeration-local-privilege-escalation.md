# Manual Enumeration: Local Privilege Escalation

This cover various methods to get situational awareness on a system.

#### Information to be obtained <a href="#information-to-be-obtained" id="information-to-be-obtained"></a>

* Username and hostname
* Group memberships of the current user
* Existing users and groups
* Operating system, version and architecture
* Network information
* Installed applications
* Running processes

#### whoami <a href="#whoami" id="whoami"></a>

Copy

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Users\root> whoami
desktop-1btki7c\root
PS C:\Users\root>
```

#### whoami /groups <a href="#whoami-groups" id="whoami-groups"></a>

```
PS C:\Users\root> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                                    Type             SID          Attributes
============================================================= ================ ============ ===============================================================
Everyone                                                      Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114    Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                                        Alias            S-1-5-32-544 Mandatory group, Enabled by default, Enabled group, Group owner
```

#### net users <a href="#net-users" id="net-users"></a>

```
C:\Users\root>net users

User accounts for \\DESKTOP-1BTKI7C

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
root                     WDAGUtilityAccount
The command completed successfully.
```

#### Get-LocalUser <a href="#get-localuser" id="get-localuser"></a>

```
PS C:\Users\root> get-localUser

Name               Enabled Description
----               ------- -----------
Administrator      False   Built-in account for administering the computer/domain
DefaultAccount     False   A user account managed by the system.
Guest              False   Built-in account for guest access to the computer/domain
root               True
WDAGUtilityAccount False   A user account managed and used by the system for Windows Defender Application Guard scenarios.
```

> **Apart from the non-standard groups, there are several built-in groups we should analyze, such as Administrators, Backup Operators, Remote Desktop Users, and Remote Management Users**

#### Get-LocalGroupMember <a href="#get-localgroupmember" id="get-localgroupmember"></a>

```
PS C:\Users\root> Get-LocalGroupMember "Users"

ObjectClass Name                             PrincipalSource
----------- ----                             ---------------
Group       NT AUTHORITY\Authenticated Users Unknown
Group       NT AUTHORITY\INTERACTIVE         Unknown
```

#### systeminfo <a href="#systeminfo" id="systeminfo"></a>

```
PS C:\Users\root> systeminfo

Host Name:                 DESKTOP-1BTKI7C
OS Name:                   Microsoft Windows 10 Pro
OS Version:                10.0.18363 N/A Build 18363
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          root
has been detected. Features required for Hyper-V will not be displayed.
```

#### \[Environment]::Is64BitProcess <a href="#environment-is64bitprocess" id="environment-is64bitprocess"></a>

Copy

```
PS C:\Users\root> [Environment]::Is64BitProcess
True
PS C:\Users\root>
```

#### ipconfig /all <a href="#ipconfig-all" id="ipconfig-all"></a>

```
PS C:\Users\root> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : DESKTOP-1BTKI7C
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : localdomain

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : localdomain
   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-0C-29-00-25-AE
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::3c66:2eb6:fd93:4a83%4(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.23.154(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : Monday, July 17, 2023 10:24:01 PM
   Lease Expires . . . . . . . . . . : Monday, July 17, 2023 11:41:42 PM
   Default Gateway . . . . . . . . . : 192.168.23.2
   DHCP Server . . . . . . . . . . . : 192.168.23.254
   DHCPv6 IAID . . . . . . . . . . . : 117443625
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2C-42-10-CC-00-0C-29-00-25-AE
   DNS Servers . . . . . . . . . . . : 192.168.23.2
   Primary WINS Server . . . . . . . : 192.168.23.2
   NetBIOS over Tcpip. . . . . . . . : Enabled

```

#### route print <a href="#route-print" id="route-print"></a>

```
PS C:\Users\root> route print
===========================================================================
Interface List
  4...00 0c 29 00 25 ae ......Intel(R) 82574L Gigabit Network Connection
  3...b4 8c 9d ce 8e ce ......Bluetooth Device (Personal Area Network)
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0     192.168.23.2   192.168.23.154     25
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
     192.168.23.0    255.255.255.0         On-link    192.168.23.154    281
   192.168.23.154  255.255.255.255         On-link    192.168.23.154    281
   192.168.23.255  255.255.255.255         On-link    192.168.23.154    281
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link    192.168.23.154    281
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link    192.168.23.154    281
===========================================================================
Persistent Routes:
  None

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  1    331 ::1/128                  On-link
  4    281 fe80::/64                On-link
  4    281 fe80::3c66:2eb6:fd93:4a83/128
                                    On-link
  1    331 ff00::/8                 On-link
  4    281 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

#### netstat -ano <a href="#netstat-ano" id="netstat-ano"></a>

```
PS C:\Users\root> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       884
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       1132
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       1908
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       524
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1120
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       388
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       1776
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING       636
  TCP    192.168.23.154:139     0.0.0.0:0              LISTENING       4
  TCP    192.168.23.154:49887   13.107.246.58:443      CLOSE_WAIT      2908
  TCP    192.168.23.154:50071   20.198.118.190:443     ESTABLISHED     388
  TCP    192.168.23.154:50424   20.198.119.143:443     ESTABLISHED     388
  TCP    192.168.23.154:50443   20.42.65.89:443        TIME_WAIT       0
  TCP    192.168.23.154:50444   20.189.173.13:443      TIME_WAIT       0
  TCP    192.168.23.154:50445   20.189.173.13:443      TIME_WAIT       0
  TCP    192.168.23.154:50446   20.189.173.13:443      TIME_WAIT       0
  TCP    192.168.23.154:50453   20.44.10.122:443       TIME_WAIT       0
  TCP    192.168.23.154:50454   20.44.10.122:443       TIME_WAIT       0
  TCP    [::]:135               [::]:0                 LISTENING       884
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:7680              [::]:0                 LISTENING       1908
  TCP    [::]:49664             [::]:0                 LISTENING       648
  TCP    [::]:49665             [::]:0                 LISTENING       524
  TCP    [::]:49666             [::]:0                 LISTENING       1120
  TCP    [::]:49667             [::]:0                 LISTENING       388
  TCP    [::]:49668             [::]:0                 LISTENING       1776
  TCP    [::]:49673             [::]:0                 LISTENING       636
  UDP    0.0.0.0:5050           *:*                                    1132
  UDP    0.0.0.0:5353           *:*                                    1560
  UDP    0.0.0.0:5355           *:*                                    1560
  UDP    127.0.0.1:1900         *:*                                    876
  UDP    127.0.0.1:58245        *:*                                    876
  UDP    127.0.0.1:60793        *:*                                    388
  UDP    192.168.23.154:137     *:*                                    4
  UDP    192.168.23.154:138     *:*                                    4
  UDP    192.168.23.154:1900    *:*                                    876
  UDP    192.168.23.154:58244   *:*                                    876
  UDP    [::]:5353              *:*                                    1560
  UDP    [::]:5355              *:*                                    1560
  UDP    [::1]:1900             *:*                                    876
  UDP    [::1]:58243            *:*                                    876
  UDP    [fe80::3c66:2eb6:fd93:4a83%4]:1900  *:*                                    876
  UDP    [fe80::3c66:2eb6:fd93:4a83%4]:58242  *:*                                    876
```

#### Get-ItemProperty <a href="#get-itemproperty" id="get-itemproperty"></a>

```
PS C:\Users\root> Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname                                                      
displayname
-----------

Microsoft Edge
Microsoft Edge Update
Microsoft Edge WebView2 Runtime

XAMPP
Microsoft Visual C++ 2015-2022 Redistributable (x64) - 14.32.31326
Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.32.31326
Microsoft Visual C++ 2015-2022 Redistributable (x86) - 14.32.31326
Microsoft Visual C++ 2022 X86 Additional Runtime - 14.32.31326
Microsoft Visual C++ 2008 Redistributable - x86 9.0.21022



PS C:\Users\root> Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname                                                                  
displayname
-----------


Microsoft Visual C++ 2022 X64 Additional Runtime - 14.32.31326
VMware Tools
Microsoft Update Health Tools
Update for Windows 10 for x64-based Systems (KB5001716)
Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.32.31326
```

#### Get-Process <a href="#get-process" id="get-process"></a>

```
PS C:\Users\root> Get-Process                                                                                                                                                          
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    175      10     6504        908       0.14   5420   0 audiodg
     75       5     2340          0       0.02   1640   1 cmd
    260      13     7452        280       0.09   4404   1 conhost
    262      13     6140       2576       1.36   7112   1 conhost
    463      14     1692        108       0.58    420   0 csrss
    405      21     5276         84       1.63    496   1 csrss
    418      15     4116       1028       0.47   3132   1 ctfmon
    257      14     4024       1376       0.27   3164   0 dllhost
    216      17     3848        336       0.11   6192   1 dllhost
    807      36    74424      15184      12.16    956   1 dwm
   1936      72    44572      20696       4.27   1772   1 explorer
     32       5     1252          0       0.06    788   0 fontdrvhost
     32       5     1476          0       0.02    796   1 fontdrvhost
      0       0       60          8                 0   0 Idle
   1084      22     5816       2852       2.20    648   0 lsass
      0       0     1068     260852      67.66   1256   0 Memory Compression
    660      39    34744          0       0.31   3308   1 Microsoft.Photos
    103       7     2444          8       0.05    288   0 MpCopyAccelerator
    221      13     2788          0       0.09   3260   0 msdtc
   1098      39    38812       9160       1.34    980   1 msedge
    132       9     2176          0       0.03   3180   1 msedge
    316      20    13760        944       0.08   6332   1 msedge
    337      18    10956       2432       0.23   6348   1 msedge
    220      15     7932        712       0.02   6412   1 msedge
    251      14     7676        580       0.16   4836   0 msiexec
    836      91   297364      43452     293.83   2160   0 MsMpEng
    318      14     3172       1488       0.02   1008   1 MusNotifyIcon
    208      12     4060       1084       0.00   4172   0 NisSrv
    769      53    25204       4884       1.92   4412   1 OneDrive
   1102      48    76908      15856       4.14   7092   1 powershell
      0      12     9916        472       0.97     88   0 Registry
    494      23    10428       1884       0.41   5004   1 RuntimeBroker
    285      17     5772        176       0.63   5072   1 RuntimeBroker
    258      14     2684        156       0.25   5328   1 RuntimeBroker
    315      16     6636       2536       0.22   5528   1 RuntimeBroker
    213      11     2564       1996       0.06   6152   1 RuntimeBroker
    232      12     2896       1208       0.08   7036   1 RuntimeBroker
    689      44    22924        740       5.64   3404   0 SearchIndexer
   1316      87   102588          0       6.17   2908   1 SearchUI
    386      16     4064        304       0.08   4708   0 SecurityHealthService
    154      10     1716          0       0.00   6132   1 SecurityHealthSystray
    378      10     3668       2632       1.81    636   0 services
     89       6     3188          0       0.03   3684   0 SgrmBroker
    565      26    15716       4164       0.61   2000   1 ShellExperienceHost
    513      18     5812       5628       1.25   3600   1 sihost
    444      36    15464          0       0.41   5212   1 SkypeApp
    152       8     2000          0       0.03   5236   1 SkypeBackgroundHost
     53       3     1148          0       0.08    300   0 smss
    427      22     5240          0       0.20   1776   0 spoolsv
    591      28    27048       5168       1.19   4948   1 StartMenuExperienceHost
   2198     139   118772      18052     216.75    388   0 svchost
    747      34    13084       5848       7.75    500   0 svchost
    929      23     9328       3976       1.91    760   0 svchost
    214      15     2096       1012       0.11    876   0 svchost
    990      17     6564       5892       4.31    884   0 svchost
    180      13     1760          0       0.03   1112   0 svchost
    717      29    22432       4680       2.34   1120   0 svchost
   1278      52    12976       4904       1.80   1132   0 svchost
    224      12     2268          0       0.02   1332   0 svchost
    408      18    14380       3888       0.70   1400   0 svchost
    325      13     2832         88       0.09   1448   0 svchost
    515      24    15228          0       1.69   1556   0 svchost
    690     198    12972       2184       1.30   1560   0 svchost
    126      10     1544          0       0.03   1616   0 svchost
    356      14     2216          0       0.02   1624   0 svchost
    408      32     9472          0       0.86   1824   0 svchost
    190      10     6944       3288       2.70   1864   0 svchost
    335      23     6256       3288      18.67   1908   0 svchost
    380      24     3284          0       0.13   2304   0 svchost
    180      11     4060        700       0.06   2332   0 svchost
    870      37    11636       3124       0.81   3552   1 svchost
    344      16     4016       8480       0.16   4456   1 svchost
    224      12     2504       1964       0.05   6440   0 svchost
    374      12     5624          0       0.34   7132   0 svchost
   2343       0      200          0     195.53      4   0 System
    256      27     5516          0       0.20   3804   1 taskhostw
    324      18     5236          0       0.09   5924   1 taskhostw
    275      59    29972          0       9.73   2280   0 TiWorker
    134       8     1748        916       0.05   2028   0 TrustedInstaller
    165      11     2896          0       0.13   2180   0 VGAuthService
    139       8     1648          0       0.02   2172   0 vm3dservice
    137       9     1792          0       0.02   2520   1 vm3dservice
    385      22     9812       6264       1.55   2112   0 vmtoolsd
    615      27    20284       2852       7.03   5580   1 vmtoolsd
    492      22    12412       3744       0.11   3812   1 WindowsInternal.ComposableShell.Experiences.TextInput.InputApp
    156      11     1324          0       0.08    524   0 wininit
    258      13     2896       1316       0.03    560   1 winlogon
    434      22    11032       7520       3.36   3456   0 WmiPrvSE
    297      16    21564       1628       0.28   4048   0 WmiPrvSE
```

#### Get-ChildItem <a href="#get-childitem" id="get-childitem"></a>

```
PS C:\Users\root>  Get-ChildItem -Path C:\ -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue                                                                           

    Directory: C:\Program Files\Common Files\microsoft shared\ink


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/18/2019   9:45 PM          19626 ThirdPartyNotices.MSHWLatin.txt


    Directory: C:\Program Files\VMware\VMware Tools


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         8/2/2022   5:37 AM         301924 open_source_licenses.txt


    Directory: C:\Program Files\Windows Defender


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/18/2019   9:43 PM           1091 ThirdPartyNotices.txt
```

Copy

```
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

#### Get-History AND Get-PSReadlineOption <a href="#get-history-and-get-psreadlineoption" id="get-history-and-get-psreadlineoption"></a>

```
PS C:\Users\root> get-history                                                                                                                                                          
  Id CommandLine
  -- -----------
   1 whoami
   2 whoami /gruops
   3 whoami /groups
   4 get-localusers
   5 get-localUser
   6 Get-LocalGroup
   7 get-localgroupmenber
   8 get-localgroupmenbers
   9 get-localgroupmembers
  10 get-localgroupmember
  11 Get-LocalGroupMember Remote Desktop Users
  12 Get-LocalGroupMember "Remote Desktop Users"
  13 Get-LocalGroupMember "Users"

PS C:\Users\root> (Get-PSReadlineOption).HistorySavePath                                                                                                                               C:\Users\root\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
PS C:\Users\root> type C:\Users\root\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt                                                                   whoami
whoami /gruops
whoami /groups
get-localusers
get-localUser
Get-LocalGroup
get-localgroupmenber
get-localgroupmenbers
get-localgroupmembers
get-localgroupmember
Get-LocalGroupMember Remote Desktop Users
Get-LocalGroupMember "Remote Desktop Users"
Get-LocalGroupMember "Users"
systeminfo
[Environment]::Is64BitProcess
ipconfig /all
route print
netstat -ano
```

### Donot forget to check Event Viewer <a href="#donot-forget-to-check-event-viewer" id="donot-forget-to-check-event-viewer"></a>
