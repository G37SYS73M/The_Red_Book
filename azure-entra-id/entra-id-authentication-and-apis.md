# Entra ID - Authentication and APIs

Microsoft Entra ID (formerly Azure AD) uses the **Microsoft Identity Platform** to handle authentication and authorization via modern protocols like **OpenID Connect (OIDC)** for authentication and **OAuth 2.0** for authorization. It also supports **SAML 2.0** for single sign-on (SSO) and legacy protocols such as **LDAP**, **Kerberos Constrained Delegation**, and **Header-based authentication** for hybrid and on-premises environments, ensuring secure access across modern and legacy systems.

![](https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F3d7e6138-63c1-42d7-94c9-f77ae6717fdb%2Fac03dd4f-922f-4f45-97f5-65b2b4cdaadd%2Fimage.png\&width=768\&dpr=4\&quality=100\&sign=381ce6e0\&sv=2)image.png

![](https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F3d7e6138-63c1-42d7-94c9-f77ae6717fdb%2Fe4673fda-351e-43d4-81ec-6725950fe701%2Fimage.png\&width=768\&dpr=4\&quality=100\&sign=456d504d\&sv=2)Basic Sign-in using OAuth

Basic Sign-in using OAuth

![](https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F3d7e6138-63c1-42d7-94c9-f77ae6717fdb%2F3c600a67-8a9d-4d31-b04f-7af4efd43048%2Fimage.png\&width=768\&dpr=4\&quality=100\&sign=24b68d46\&sv=2)Basic Sign-in with token acquisition

Basic Sign-in with token acquisition

#### Tokens <a href="#tokens" id="tokens"></a>

* OAuth 2.0 and OIDC use bearer tokens which are JSON Web Tokens.
* A bearer token, as the name suggests, grants the bearer access to a protected resource.
* There are three types of tokens used in OIDC:
  * **Access Tokens** - The client presents this token to the resource server to access resources. It can be used only for a specific combination of user, client, and resource and cannot be revoked until expiry - that is 1 hour by default.
  * **ID Tokens** - The client receives this token from the authorization server. It contains basic information about the user. It is bound to a specific combination of user and client.
  * **Refresh Tokens** - Provided to the client with access token. Used to get new access and ID tokens. It is bound to a specific combination of user and client and can be revoked. Default expiry is 90 days for inactive refresh tokens and no expiry for active tokens.

**Using Token with CLI tools - Az Powershell**

* Both Az PowerShell and AzureAD modules allow the use of Access tokens for authentication.
* Usually, tokens contain all the claims (including that for MFA and Conditional Access etc.) so they are useful in bypassing such security controls.
* If you are already connected to a tenant, request an access token for resource manager (ARM)

```powershell
Get-AzAccessToken
(Get-AzAccessToken).Token
```

* Request an access token for Microsoft Graph to access Entra ID. Supported tokens - `AadGraph`, `AnalysisServices`, `Arm`, `Attestation`, `Batch`, `DataLake`, `KeyVault`, `MSGraph`, `OperationalInsights`, `ResourceManager`, `Storage, Synapse`

```powershell
Get-AzAccessToken -ResourceTypeName MSGraph
```

* From older versions of Az PowerShell, get a token for Microsoft Graph

```powershell
(Get-AzAccessToken -Resource "<https://graph.microsoft.com>").Token
```

* Use the access token

```powershell
Connect-AzAccount -AccountId test@defcorphq.onmicrosoft.com -AccessToken <AzureAccessToken>
```

* Use other access tokens. In the below command, use the one for MSGraph (access token is still required) for accessing Entra ID

```powershell
Connect-AzAccount -AccountId test@defcorphq.onmicrosoft.com -AccessToken <AzureAccessToken> -MicrosoftGraphAccessToken <GraphAccessToken>
```

**Using Tokens with CLI tools - az cli**

* az cli can request a token but cannot use it! (Actually you can, see the next slide)
* Request an access token (ARM)

```powershell
az account get-access-token
```

* Request an access token for `aad-graph`. Supported tokens - `aad-graph`, `arm`, `batch`, `data-lake`, `media`, `ms-graph`, `oss-rdbms`.

```powershell
az account get-access-token --resource-type ms-graph
```

**Using Tokens with APIs - Management**

The two REST APIs endpoints that are most widely used are

* Azure Resource Manager - [management.azure.com](http://management.azure.com/)
* Microsoft Graph - [graph.microsoft.com](http://graph.microsoft.com/) (AADGraph which is deprecated is [graph.windows.net](http://graph.windows.net/))
* Let's have a look at super simple PowerShell codes for using the APIs

```powershell
# Get an access token and use it with ARM API. For example, list all the subscriptions
$Token = 'eyJ0eXAi..'
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

```powershell
# Get an access token for MS Graph. For example, list all the users

$Token = 'eyJ0eXAi..'
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

#### Stealing Tokens <a href="#stealing-tokens" id="stealing-tokens"></a>

**az cli**

* az cli (before 2.30.0 – January 2022) stores access tokens in clear text in `accessTokens.json` in the directory `C:\\Users\\[username]\\.Azure`
* We can read tokens from the file, use them and request new ones too!
* `azureProfile.json` in the same directory contains information about subscriptions.
* You can modify `accessTokens.json` to use access tokens with az cli but better to use with Az PowerShell module.
* To clear the access tokens, always use az logout

**Az PowerShell**

* Az PowerShell (older versions) stores access tokens in clear text in `TokenCache.dat` in the directory `C:\\Users\\[username]\\.Azure`
* It also stores `ServicePrincipalSecret` in clear-text in `AzureRmContext.json` if a service principal secret is used to authenticate.
* Another interesting method is to take a process dump of PowerShell and looking for tokens in it!
* Users can save tokens using `Save-AzContext`, look out for them!
* Search for `Save-AzContext` in PowerShell console history!
* Always use `Disconnect-AzAccount`!!
* AzureAD module cannot request a token but can use one for `AADGraph`or Microsoft Graph!
* Use the AAD Graph token

```powershell
Connect-AzureAD -AccountId test@defcorphq.onmicrosoft.com -AadAccessToken $token
```

* Use the MS Graph token with Mg module

```powershell
Connect-MgGraph –AccessToken ($Token | ConvertToSecureString -AsPlainText -Force)
```

* Some places to look or fetch tokens

```powershell
# Powershell history file location can be seen and changed here
Get-PSReadLineOption

# Read access token
(Get-AzAccessToken).Token

# Use to Token to bypass CAP
$token = (Get-AzAccessToken).Token
Connect-AzAccount -AccountId test@defcorphq.onmicrosoft.com -AccessToken $token
Get-AzVM

$msgraphaccesstoken = (Get-AzAccessToken -ResourceTypeName MSGraph).Token
Connect-AzAccount -AccountId test@defcorphq.onmicrosoft.com -AccessToken $token -MicrosoftGraphAccessToken $msgraphaccesstoken
Get-AzADUsers
```

#### Continuous Access Evaluation (CAE) <a href="#continuous-access-evaluation-cae" id="continuous-access-evaluation-cae"></a>

* CAE can help in invalidating access tokens before their expiry time (default 1 hour).
* Useful in cases like
  * User termination or password change enforced in near real time
  * Blocks use of access token outside trusted locations when location based conditional access is present
* In CAE sessions, the access token lifetime is increased up to 28 hours.

**CAE – Claims Challenge**

* CAE needs the client (Browser/App/CLI) to be CAE-capable – the client must understand that the token has not expired but cannot be used.
* Outlook, Teams, Office (other than web) and OneDrive web apps and apps support CAE.
* The xms\_cc claim with a value of “CP1" in the access token is the authoritative way to identify a client application is capable of handling a claims challenge.
* In case the client is not CAE-capable, a normal access token with 1 hour expiry is issued.
* We can find `xms_cc` claim in MSGraph token `testuser` that was requested using Az PowerShell.

> **Access tokens issued for managed identities by Azure IMDS are not CAE-enabled.**

**Working Scenarios**

1. **Critical event evaluation**
   * User account is deleted or disabled
   * Password change or reset for a user
   * MFA enabled for a user
   * Refresh token is revoked
   * High user risk detected by Entra ID Protection (not supported by SharePoint online)
   * Only Exchange Online, SharePoint online and Teams are supported.
2. Conditional Access policy evaluation
   * Only IP-based (both IPv4 and IPv6) named locations are supported. Other location conditions like MFA trusted IPs or country-based locations are not supported.
   * Exchange Online, SharePoint online, Teams and MS Graph are supported.
