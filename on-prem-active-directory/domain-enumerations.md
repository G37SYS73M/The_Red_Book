---
description: Using PowerView.ps1 and the AD Module
---

# Domain Enumerations

### Checking for Domain Connections <a href="#checking-for-domain-connections" id="checking-for-domain-connections"></a>

```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()


net user /domain
net user atlante.phillis /domain

net group /domain
net group "IT Admins" /domain
```

### Current Domain <a href="#current-domain" id="current-domain"></a>

```powershell
Get-NetDomain

Get-ADDomain
```

### Another Domain: <a href="#another-domain" id="another-domain"></a>

```powershell
Get-NetDomain -Domain name

Get-ADDomain -Identity name
```

### Domain SID: <a href="#domain-sid" id="domain-sid"></a>

```powershell
Get-DomainSID
```

### Domain Policy <a href="#domain-policy" id="domain-policy"></a>

```powershell
Get-DomainPolicy
```

### Domain Controller <a href="#domain-controler" id="domain-controler"></a>

```powershell
Get-NetDomainController
Get-NetDomainController -Domain name

Get-AdDomainController
```

### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

```powershell
Get-NetUser
Get-NetUser -Username name
Get-NetUser | select * (any property)

get-aduser -Filter * -property *
get-aduser -identity name -property *

Get-userproperty #not working
Get-userProperty -Properties

get-aduser -filter -properties * | select * | select -first 1 | getmember -membertype * prperty |select name


very Importtant

Find-UserField -SearchField Description -SearchTerm "built"

get-aduser -Filter 'Description -like "*bulid"' -Properties Description | select name,Description
```

### Computer Enum: <a href="#computer-enum" id="computer-enum"></a>

```powershell
get-netComputer
get-netcomputer -operatingsystem "*Server 2016*"
get-netcomputer -Ping
get-netComputer -FullData


get-adcomputer -Filter * | Select name
get-adcomputer -Filter 'operatingSystem -like "*server 2016*"' - properties operatingsystem | select name,operating,system
get-adcomputer -filter * - properties DNSHostName | %{Test-Connection -count 1 -computername $_.DNSHostName}
get-adcomputer -Filter * -Properties 
```

### Group Enum <a href="#group-enum" id="group-enum"></a>

```powershell
get-netgroup
get-netgroup -domain <target>
get-netgroup -Fulldata
get-netgroup *admin*
get-netgroupmember -GroupName "domain Admins" - Recurse
get-netgroup -username "student1"

get-adgroup -filter *  | select name
get-adgroup -filter * -Properties
get-adGroup -Filter 'Nmae -like "*admin*"' |select Name
get-adgroupmember -identity "Domain admins" -recursive
get-adprincipalgroupmembership -dentity student1

for local groups
get-netlocalgroup -computername {} -listgroup
get-netlocalgroup -computername {} -recurse
```

### Logon check <a href="#logon-check" id="logon-check"></a>

```powershell
get-netloggedon -computername <severname>
get-loggedonlocal -computername <severname>
get-lastloggedon -computername
```

### Share enum important files <a href="#share-enum-important-files" id="share-enum-important-files"></a>

```powershell
Find-DomainShare -CheckShareAccess

Invoke-sharefinder -verbose        

Invoke-filefinder -verbose

Get-netfileserver 
```

### GPO Enumerations <a href="#gpo-enumerations" id="gpo-enumerations"></a>

```powershell
get-netGPO
get-netGPO -Computername

get-netGPOGroup

gpresult /R

find-gpocomputeradmin -computername

find-gpolocation -usernmae <> -verbose
```

### OU Enums <a href="#ou-enums" id="ou-enums"></a>

```powershell
get-netou -fulldata

get-netgop -gponame
```

### ACL enumeration <a href="#acl-enumeration" id="acl-enumeration"></a>

```powershell
get-objectAcl -SamAccountname <> -ResolveGUIDS

getObjectAcl -ADSprefix 'CN=Administrator,CN=Users' -verbose

get-ObjectACL -ADSpath "LDAP://CN=Domain Admins,CN=Users,DC=Dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose


#searchs for intersting ACL
invoke-ACLScanner -ResolveGUIDs 


get-pathAcl -Path "\\dcorp-dc.doolarcorp.moneycorp.local\sysvol"

Convert-SidToName
```

### Domain Trust Enumeration <a href="#domain-trust-enumeration" id="domain-trust-enumeration"></a>

* one way trust
* Two Way trust

```powershell
get-netdomaintrust
get-netdomaintrust -Domain 

get-adtrust
get-adtrust -Identity 
```

### Forest Enumeration <a href="#forest-enumeration" id="forest-enumeration"></a>

```powershell
get-netForest
get-netforest -Forest

get-ADforest
get-ADforest -identity

# all domain in current forest
get-netforestdomain
get-netforestdomain -Forest 
```

### Forest trust <a href="#forest-trust" id="forest-trust"></a>

```powershell
get-netforestcatalog
get-netforestcatalog -forest

get-adforest | select -Expandproperty GlobalCatalog


get-netforesttrust
get-netforesttrust -forest 

get-adtrust -filter 'msDS-TrustforestTrustInfo -ne "$null"'
```

### User Hunting <a href="#user-hunting" id="user-hunting"></a>

The permissions required to enumerate sessions with **NetSessionEnum** are defined in the **SrvsvcSessionInfo** registry key, which is located in the **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity**

```powershell
#PowerView 

Find-LocaladminAccess -verbose
Get-NetComputer           -> IMPT

#command uses the NetWkstaUserEnum and NetSessionEnum APIs under the hood
Get-NetSession

Invoke-CheckLocalAdminAccess

#use FIND-wmiLocalAdminAccess.ps1

Invoke-EnumarateLocalAdmin -Vervose // uses Get-NetLocalGroup

#Important Uses Get-NetGroupMember
Invoke-UserHunter
Invoke-UserHunter -Stealth
Invoke-UserHunter -GroupName "RDPUsers"



#Confirm admin access
Invoke-UserHunter -CheckAccess



```
