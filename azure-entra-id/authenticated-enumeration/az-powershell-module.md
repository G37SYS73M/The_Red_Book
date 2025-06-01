# Az PowerShell Module

```powershell
Install-Module Az
```

1. Connecting

```powershell
Connect-AzAccount

$passwd = ConvertTo-SecureString "V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential("test@defcorphq.onmicrosoft.com",$passwd)
Connect-AzAccount -Credential $creds
```

1. Searching for commands

```powershell
Get-Command *azad*
Get-Command *az*
Get-Command *azvm*
```

**Basics**

* Get the information about the current context (Account, Tenant, Subscription etc.)

```powershell
Get-AzContext
```

* List all available contexts

```powershell
Get-AzContext -ListAvailable
```

> If we enumerate we can use `Set-AzContext -Context <Active Context>`

* Enumerate subscriptions accessible by the current user

```powershell
Get-AzSubscription
```

* Enumerate all resources visible to the current user

```powershell
Get-AzResource
```

* Enumerate all Azure RBAC role assignments

```powershell
Get-AzRoleAssignment
```

* Virtual Machines

```powershell
 Get-AzVM
```

* App Services

```powershell
Get-AzWebApp | Select-Object ResourceGroup, Name, Location, State
```

* Function Apps

```powershell
Get-AzWebApp | Select-Object ResourceGroup, Name, Kind, Location, State
```

* Storage Accounts

```powershell
Get-AzStorageAccount | fl *
```

* Key Vaults

```powershell
Get-AzKeyVault | fl *
```

**Users**

* Enumerate all users

```powershell
Get-AzADUser
```

* Enumerate a specific user

```powershell
Get-AzADUser -UserPrincipalName test@defcorphq.onmicrosoft.com
```

* Search for a user based on string in first characters of DisplayName (wildcard not supported)

```powershell
Get-AzADUser -SearchString "admin"
```

* Search for users who contain the word "admin" in their Display name

```powershell
Get-AzADUser |?{$_.Displayname -match "admin"}
```

**Groups**

* List all groups

```powershell
Get-AzADGroup 
```

* Enumerate a specific group

```powershell
Get-AzADGroup -ObjectId 783a312d-0de2-4490-92e4-539b0e4ee03e
```

* Search for a group based on string in first characters of DisplayName (wildcard not supported)

```powershell
Get-AzADGroup -SearchString "admin" | fl *
```

* To search for groups which contain the word "admin" in their name:

```powershell
Get-AzADGroup |?{$_.Displayname -match "admin"}
```

* Get members of a group

```powershell
Get-AzADGroupMember -ObjectId 783a312d-0de2-4490-92e4-539b0e4ee03e
```

**Apps**

* Get all the application objects registered with the current tenant (visible in App Registrations in Azure portal). An application object is the global representation of an app.

```powershell
Get-AzADApplication
```

* Get all details about an application

```powershell
Get-AzADApplication -ObjectId a1333e88-1278-41bf-8145-155a069ebed0
```

* Get an application based on the display name

```powershell
Get-AzADApplication | ?{$_*.*DisplayName -match "app"}
```

* The `Get-AzADAppCredential` will show the applications with an application password but password value is not shown. List all the apps with an application password.

```powershell
Get-AzADApplication | %{if(Get-AzADAppCredential -ObjectID $_.ID){$_}}
```

**Service Principals**

* Enumerate Service Principals (visible as Enterprise Applications in Azure Portal). Service principal is local representation for an app in a specific tenant and it is the security object that has privileges. This is the 'service account'!
* Service Principals can be assigned Azure roles.
* Get all service principals

```powershell
Get-AzADServicePrincipal 
```

* Get all details about a service principal

```powershell
Get-AzADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264
```

* Get a service principal based on the display name

```powershell
Get-AzADServicePrincipal | ?{$_.DisplayName -match "app"}
```
