# Application Proxy

* Application Proxy allows access to on-prem web applications after sign-in to Entra ID.
* Application proxy has following components
  * Endpoint - This is the external URL that the users browse to access the on-prem application. External users must authenticate to AAD
  * Application Proxy Service - This services runs in the cloud and passes the token provided by Entra ID to the on-prem connector
  * Application Proxy Connector - This is an agent that runs on the on-prem infrastructure and acts as a communication agent between the cloud proxy service and on-prem application. It also communicates with the on-prem AD in case of SSO
  * On-prem application - The application that is exposed using application prox
* Compared to directly exposing an on-prem app, application proxy does provide additional security (authentication handled by Entra ID, Conditional Access etc.)
* But, it does NOT help if the on-prem application has code or deployment related vulnerabilities.
* We can enumerate the applications that has application proxy configured using the Azure AD module (may take a few minutes to complete).

```powershell
. C:\\AzAD\\Tools\\Get-MgApplicationProxyApplication.ps1
```

* Get the Service Principal (Enterprise Application)

```powershell
Get-MgServicePrincipal -Filter "DisplayName -eq 'Finance Management System'"
```

* Use Get-MgApplicationProxyAssignedUsersAndGroups.ps1 to find users and groups assigned to the application

```powershell
. C:\\AzAD\\Tools\\Get-MgApplicationProxyAssignedUsersAndGroups.ps1
Get-ApplicationProxyAssignedUsersAndGroups -ObjectId ec350d24-e4e4-4033-ad3f-bf60395f0362
```

\
