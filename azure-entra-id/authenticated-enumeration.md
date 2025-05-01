# Authenticated Enumeration

### Connect to the tenant using the Az PowerShell module <a href="#connect-to-the-tenant-using-the-az-powershell-module" id="connect-to-the-tenant-using-the-az-powershell-module"></a>

```powershell
$passwd = ConvertTo-SecureString "V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("test@defcorphq.onmicrosoft.com", $passwd)
Connect-AzAccount -Credential $creds
```

### Connecting to the Microsoft Graph using <a href="#connecting-to-the-microsoft-graph-using" id="connecting-to-the-microsoft-graph-using"></a>

```powershell
$Token = (Get-AzAccessToken -ResourceTypeName MSGraph).Token
Connect-MgGraph -AccessToken ($Token | ConvertTo-SecureString -AsPlainText -Force)
```

### To enumerate all users <a href="#to-enumerate-all-users" id="to-enumerate-all-users"></a>

```powershell
Get-MgUser -All
```

#### To list only the UPNs of the users <a href="#to-list-only-the-upns-of-the-users" id="to-list-only-the-upns-of-the-users"></a>

```powershell
Get-MgUser -All | select UserPrincipalName
```

### To list all the groups <a href="#to-list-all-the-groups" id="to-list-all-the-groups"></a>

```powershell
Get-MgGroup -All
```

### list all the devices <a href="#list-all-the-devices" id="list-all-the-devices"></a>

```powershell
Get-MgDevice
```

### To get all the Global Administrators <a href="#to-get-all-the-global-administrators" id="to-get-all-the-global-administrators"></a>

```powershell
$RoleId = (Get-MgDirectoryRole -Filter "DisplayName eq 'Global Administrator'").Id
(Get-MgDirectoryRoleMember -DirectoryRoleId $RoleId).AdditionalProperties
```

### To list all custom directory roles <a href="#to-list-all-custom-directory-roles" id="to-list-all-custom-directory-roles"></a>

```powershell
Get-MgRoleManagementDirectoryRoleDefinition | ?{$_.IsBuiltIn -eq $False} | select DisplayName
```

### Enumeration Using AzModule <a href="#enumeration-using-azmodule" id="enumeration-using-azmodule"></a>

#### Connecting Using AzModule <a href="#connecting-using-azmodule" id="connecting-using-azmodule"></a>

```powershell
$passwd = ConvertTo-SecureString "V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r" -AsPlainText -Force

$creds = New-Object System.Management.Automation.PSCredential ("test@defcorphq.onmicrosoft.com", $passwd)

Connect-AzAccount -Credential $creds
```

List all the resources accessible to the current account:

```powershell
Get-AzResource
```

Get all the role assignments for the test user:

```powershell
Get-AzRoleAssignment -SignInName test@defcorphq.onmicrosoft.com
```

list all the VMs where the current user has at least the Reader role:

```powershell
Get-AzVM | fl
```

List all App Services

```powershell
Get-AzWebApp | ?{$_.Kind -notmatch "functionapp"}
```

To list Function Apps

```powershell
Get-AzFunctionApp
```

List storage accounts:

```powershell
Get-AzStorageAccount | fl
```

list the readable keyvaults for the current user

```powershell
Get-AzKeyVault
```

### Enumeration Using az cli <a href="#enumeration-using-az-cli" id="enumeration-using-az-cli"></a>

Connecting

```batch
az login -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r
```

list all the VMs where the current user has at least the Reader role.

```batch
az vm list 
```

listing the 'name' of the VMs

```batch
az vm list --query "[].[name]" -o table
```

the names of app services

```batch
az webapp list --query "[].[name]" -o table
```

list Function Apps

```batch
az functionapp list --query "[].[name]" -o table
```

list storage accounts

```batch
az storage account list
```

readable keyvaults for the current user

```batch
az keyvault list
```

### Enumeration using ROADTools <a href="#enumeration-using-roadtools" id="enumeration-using-roadtools"></a>

```batch
cd C:\AzAD\Tools\ROADTools

.\venv\Scripts\activate


roadrecon auth -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r

roadrecon gather

roadrecon gui

```

**Enumerating Conditional Access Policies**

Note that it is possible to enumerate Conditional Access Policies as a normal user using RoadRecon. This is due to the “internal-1.61” AAD Graph API version.bat

```batch
roadrecon plugin policies
```

Open caps.html (from C:\AzAD\Tools\ROADTools)to find Conditional Access Policies in the target environment:

### Enumeration using StormSpotter <a href="#enumeration-using-stormspotter" id="enumeration-using-stormspotter"></a>

```batch
cd C:\AzAD\Tools\stormspotter\backend\
pipenv shell


cd C:\AzAD\Tools\stormspotter\frontend\dist\spa\
quasar.cmd serve -p 9091 --history


cd C:\AzAD\Tools\stormspotter\stormcollector\
pipenv shell


az login -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r

python C:\AzAD\Tools\stormspotter\stormcollector\sscollector.pyz cli

```

### Enumeration using BloodHound <a href="#enumeration-using-bloodhound" id="enumeration-using-bloodhound"></a>

```batch
$passwd = ConvertTo-SecureString "V3ryH4rdt0Cr4ckN0OneC@nGu355ForT3stUs3r" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("test@defcorphq.onmicrosoft.com", $passwd)
Connect-AzAccount -Credential $creds


Import-Module C:\AzAD\Tools\AzureAD\AzureAD.psd1
Connect-AzureAD -Credential $creds

. C:\AzAD\Tools\AzureHound\AzureHound.ps1
Invoke-AzureHound -Verbose
```

[\
](https://g37sys73m.gitbook.io/g37sys73ms-manual/azure-entra-id/entra-id-authentication-and-apis)
