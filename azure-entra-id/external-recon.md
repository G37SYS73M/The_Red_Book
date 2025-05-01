# External Recon

### Azure Tenant <a href="#azure-tenant" id="azure-tenant"></a>

#### Get if Azure tenant is in use, tenant name and Federation <a href="#get-if-azure-tenant-is-in-use-tenant-name-and-federation" id="get-if-azure-tenant-is-in-use-tenant-name-and-federation"></a>

```
https://login.microsoftonline.com/getuserrealm.srf?login=[USERNAME@ValidDOMAIN]&xml=1
```

#### Get the Tenant ID <a href="#get-the-tenant-id" id="get-the-tenant-id"></a>

```
https://login.microsoftonline.com/[DOMAIN]/.well-known/openid-configuration
```

#### Validate Email ID by sending POST requests to <a href="#validate-email-id-by-sending-post-requests-to" id="validate-email-id-by-sending-post-requests-to"></a>

```
https://login.microsoftonline.com/common/GetCredentialType
```

#### We can use the AADInternals tool to gather information <a href="#we-can-use-the-aadinternals-tool-to-gather-information" id="we-can-use-the-aadinternals-tool-to-gather-information"></a>

```powershell
Import-Module C:\AzAD\Tools\AADInternals\AADInternals.psd1

Get-AADIntLoginInformation -UserName admin@defcorphq.onmicrosoft.com
```

#### To get the Tenant ID <a href="#to-get-the-tenant-id" id="to-get-the-tenant-id"></a>

```powershell
Get-AADIntTenantID -Domain defcorphq.onmicrosoft.com
```

#### Get tenant domains <a href="#get-tenant-domains" id="get-tenant-domains"></a>

```powershell
Get-AADIntTenantDomains -Domain defcorphq.onmicrosoft.com
Get-AADIntTenantDomains -Domain deffin.onmicrosoft.com
Get-AADIntTenantDomains -Domain microsoft.com
```

#### Get all the information (as external) <a href="#get-all-the-information-as-external" id="get-all-the-information-as-external"></a>

```powershell
Invoke-AADIntReconAsOutsider -DomainName defcorphq.onmicrosoft.com
```

#### Email IDs <a href="#email-ids" id="email-ids"></a>

* We can use o365creeper ([https://github.com/LMGsec/o365creeper](https://github.com/LMGsec/o365creeper)) to check if an email ID belongs to a tenant.
* It makes requests to the `GetCredentialType` API.

```batch
C:\Python27\python.exe C:\AzAD\Tools\o365creeper\o365creeper.py -f C:\AzAD\Tools\emails.txt -o C:\AzAD\Tools\validemails.txt
```

#### Azure Services <a href="#azure-services" id="azure-services"></a>

* Azure services are available at specific domains and subdomains. We can enumerate if the target organization is using any of the services by looking for such subdomains.
* The tool that we will use for this is MicroBurst ([https://github.com/NetSPI/MicroBurst](https://github.com/NetSPI/MicroBurst))
* Microburst is a useful tool for security assessment of Azure. It uses `Az`, `AzureAD`, `AzurRM` and `MSOL` tools and additional REST API calls.

```powershell
Import-Module C:\AzAD\Tools\MicroBurst\MicroBurst.psm1 -Verbose
```

Enumerate all subdomains for an organization specified using the '-Base' parameter:

```powershell
Invoke-EnumerateAzureSubDomains -Base defcorphq -Verbose
```

#### To Validate Emails we will use o365creeper <a href="#to-validate-emails-we-will-use-o365creeper" id="to-validate-emails-we-will-use-o365creeper"></a>

```powershell
C:\Python27\python.exe C:\AzAD\Tools\o365creeper\o365creeper.py -f C:\AzAD\Tools\emails.txt 
```
