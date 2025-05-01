# Password Spray/Brute-Force

**Password Spray Attack**: We will use a single password across multiple enumerated users to attempt unauthorized access.

**Risks**: This method is **noisy** and can lead to detection due to multiple failed login attempts.

**Azure Attack Targets**: In Azure, password spray attacks can target various **API endpoints** such as **Azure AD Graph**, **Microsoft Graph**, **Office 365 Reporting Webservice**, and others, making it possible to exploit different services within the Azure environment.

* We can use `MSOLSpray` ([https://github.com/dafthack/MSOLSpray](https://github.com/dafthack/MSOLSpray)) for password spray against the accounts that we discovered.
* The tool supports `fireprox` ([https://github.com/ustayready/fireprox](https://github.com/ustayready/fireprox)) to rotate source IP address on auth request.



```powershell
. C:\AzAD\Tools\MSOLSpray\MSOLSPray.ps1 

Invoke-MSOLSpray -UserList C:\AzAD\Tools\validemails.txt -Password SuperVeryEasytoGuessPassword@1234 -Verbose
```

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FU3V9Nv8FTPW9CA1BXFKf%252Fimage.png%3Falt%3Dmedia%26token%3D19401186-7acb-46a7-be04-890d373e724b&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=b8a3b98d&#x26;sv=2" alt=""><figcaption></figcaption></figure>
