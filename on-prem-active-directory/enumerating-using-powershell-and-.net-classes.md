# Enumerating using PowerShell and .NET Classes

#### LDAP path format <a href="#ldap-path-format" id="ldap-path-format"></a>

```powershell
LDAP://HostName[:PortNumber][/DistinguishedName]
```

#### To invoke the Domain Class and the GetCurrentDomain method, we’ll run the following command in PowerShell: <a href="#to-invoke-the-domain-class-and-the-getcurrentdomain-method-well-run-the-following-command-in-powersh" id="to-invoke-the-domain-class-and-the-getcurrentdomain-method-well-run-the-following-command-in-powersh"></a>

```aspnet
> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()


Forest                  : evilcorp.local
DomainControllers       : {WIN-12S5Q40APQO.evilcorp.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : WIN-12S5Q40APQO.evilcorp.local
RidRoleOwner            : WIN-12S5Q40APQO.evilcorp.local
InfrastructureRoleOwner : WIN-12S5Q40APQO.evilcorp.local
Name                    : evilcorp.local
```

#### Get A Value of a particular property <a href="#get-a-value-of-a-particular-property" id="get-a-value-of-a-particular-property"></a>

```powershell
> $obj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
> $obj.DomainControllers


Forest                     : evilcorp.local
CurrentTime                : 7/24/2023 6:30:26 AM
HighestCommittedUsn        : 13312
OSVersion                  : Windows Server 2019 Standard
Roles                      : {SchemaRole, NamingRole, PdcRole, RidRole...}
Domain                     : evilcorp.local
IPAddress                  : fe80::c4bc:8a08:6a1d:fc69%6
SiteName                   : Default-First-Site-Name
SyncFromAllServersCallback :
InboundConnections         : {}
OutboundConnections        : {}
Name                       : WIN-12S5Q40APQO.evilcorp.local
Partitions                 : {DC=evilcorp,DC=local, CN=Configuration,DC=evilcorp,DC=local, CN=Schema,CN=Configuration,DC=evilcorp,DC=local, DC=DomainDnsZones,DC=evilcorp,DC=local...}



> $obj.DomainControllers.name

WIN-12S5Q40APQO.evilcorp.local
```

#### We can use ADSI directly in PowerShell to retrieve the DN. We’ll use two single quotes to indicate that the search starts at the top of the AD hierarchy. <a href="#we-can-use-a-dsi-directly-in-powershell-to-retrieve-the-dn.-well-use-two-single-quotes-to-indicate-t" id="we-can-use-a-dsi-directly-in-powershell-to-retrieve-the-dn.-well-use-two-single-quotes-to-indicate-t"></a>

```powershell
> ([adsi]'').distinguishedName

DC=evilcorp,DC=local
```

#### The final script generates the LDAP shown below. Note that in order to clean it up, we have removed the comments. Since we only needed the PdcRoleOwner property’s name value from the domain object, we add that directly in our $PDC variable on the first line, limiting the amount of code required: <a href="#the-final-script-generates-the-ldap-shown-below.-note-that-in-order-to-clean-it-up-we-have-removed-t" id="the-final-script-generates-the-ldap-shown-below.-note-that-in-order-to-clean-it-up-we-have-removed-t"></a>

**Final Script**

```powershell
$domainobj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDCName = $domainObj.PdcRoleOwner.Name
$DC = ([adsi]'').distinguishedName

#Final LDAP Query
$LDAPquery = "LDAP://$PDCName/$DC"
$LDAPquery
```

**Output**

```powershell
> C:\Users\Administrator\Desktop\Enumaration\Enum-1.ps1
LDAP://WIN-12S5Q40APQO.evilcorp.local/DC=evilcorp,DC=local
```

#### Adding Search Functionality <a href="#adding-search-functionality" id="adding-search-functionality"></a>

We will use two **.NET** classes that are located in the **System.DirectoryServices** namespace, more specifically the **DirectoryEntry** and **DirectorySearcher** classes. Let’s discuss these before we implement them.

_**One thing to note with DirectoryEntry is that we can pass it credentials to authenticate to the domain.**_

The **DirectorySearcher** class performs queries against AD using **LDAP**. When creating an instance of **DirectorySearcher**, we must specify the AD service we want to query in the form of the **SearchRoot** property. Since the **DirectoryEntry** class encapsulates the **LDAP** path that points to the top of the hierarchy, we will pass that as a variable to **DirectorySearcher.** The **DirectorySearcher** documentation lists **FindAll()**, which returns a collection of all the entries found in AD.

**Final Code**

```powershell
$domainobj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDCName = $domainObj.PdcRoleOwner.Name
$DC = ([adsi]'').distinguishedName

#Final LDAP Query
$LDAPquery = "LDAP://$PDCName/$DC"

#Searching
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAPquery)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
$dirsearcher.FindAll()
```

The official documentation reveals different values of the **samAccountType** attribute, but we’ll start with **0x30000000 (decimal 805306368)**, which will enumerate all users in the domain. To implement the filter in our script, we can simply add the filter to the **$dirsearcher.filter** as shown below:

```powershell
$domainobj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDCName = $domainObj.PdcRoleOwner.Name
$DC = ([adsi]'').distinguishedName

#Final LDAP Query
$LDAPquery = "LDAP://$PDCName/$DC"

#Searching
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAPquery)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)

#Filtering
$dirsearcher.filter="samAccountType=805306368"

$dirsearcher.FindAll()
```

#### Printing Info Of a Principal <a href="#printing-info-of-a-principal" id="printing-info-of-a-principal"></a>

```powershell
$domainobj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDCName = $domainObj.PdcRoleOwner.Name
$DC = ([adsi]'').distinguishedName

#Final LDAP Query
$LDAPquery = "LDAP://$PDCName/$DC"

#Searching
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAPquery)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)


#Filtering
$dirsearcher.filter="name=ediva sharline"

$result = $dirsearcher.FindAll()

Foreach($obj in $result)
{
 Foreach($prop in $obj.Properties)
 {
    $prop.memberof
 }
 Write-Host "-------------------------------"
}
```

#### Creating a function with arguments <a href="#creating-a-function-with-arguments" id="creating-a-function-with-arguments"></a>

```powershell
function LDAPSearch {
 param (
 [string]$LDAPQuery
 )

 $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name

 $DistinguishedName = ([adsi]'').distinguishedName

 $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")

 $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)

 return $DirectorySearcher.FindAll()
}
```

#### Example <a href="#example" id="example"></a>

```powershell
> LDAPSearch -LDAPQuery "(name=ediva sharline)"

Path                                                                                  Properties                                      
----                                                                                  ----------                                      
LDAP://WIN-12S5Q40APQO.evilcorp.local/CN=Ediva Sharline,CN=Users,DC=evilcorp,DC=local {givenname, codepage, objectcategory, dscorep...


> foreach ($group in $(LDAPSearch -LDAPQuery "(objectCategory=group)")) {$group.properties | select {$_.cn}, {$_.member}}


$_.cn                                   $_.member                                                                                     
-----                                   ---------                                                                                     
Administrators                          {CN=Domain Admins,CN=Users,DC=evilcorp,DC=local, CN=Enterprise Admins,CN=Users,DC=evilcorp,...
Users                                   {CN=Domain Users,CN=Users,DC=evilcorp,DC=local, CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC...
Guests                                  {CN=Domain Guests,CN=Users,DC=evilcorp,DC=local, CN=Guest,CN=Users,DC=evilcorp,DC=local}      
Print Operators                                                                                                                       
Backup Operators                                                                                                                      
Replicator                                                                                                                            
Remote Desktop Users                                                                                                                  
Network Configuration Operators                                                                                                       
Performance Monitor Users                                                                                                             
Performance Log Users                                                                                                                 
Distributed COM Users                                                                                                                 
IIS_IUSRS                               CN=S-1-5-17,CN=ForeignSecurityPrincipals,DC=evilcorp,DC=local                                 
Cryptographic Operators                                                                                                               
Event Log Readers                                                                 


> $sales = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=IT Admins))"

>  $sales.properties.member

CN=Lewie Morissa,CN=Users,DC=evilcorp,DC=local
CN=Kristal Alissa,CN=Users,DC=evilcorp,DC=local
CN=Catlaina Justinn,CN=Users,DC=evilcorp,DC=local
CN=Phelia Lyssa,CN=Users,DC=evilcorp,DC=local
CN=Berni Levy,CN=Users,DC=evilcorp,DC=local
CN=Clea Kristos,CN=Users,DC=evilcorp,DC=local
CN=Sarina Kristien,CN=Users,DC=evilcorp,DC=local



```
