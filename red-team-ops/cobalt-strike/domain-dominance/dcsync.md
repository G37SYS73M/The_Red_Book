# DCSync

Domain controllers in a forest or domain frequently replicate data between themselves.  For instance, when a new user is created, the request is serviced by one domain controller (usually the closest one based on how the sites are architected).  Information about that new user is then replicated to all of the other DCs in that domain via the [Directory Replication Service](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) (DRS) protocol.

DCSync is a technique \[[T1003.006](https://attack.mitre.org/techniques/T1003/006/)] where an adversary leverages this same protocol to pull replication data, specifically usernames and password hashes, from a domain controller.  This requires access as a domain or enterprise admin, or a domain controller computer account.

This capability is built into tools such as Mimikatz's `lsadump::dcsync` command.  Beacon also provides a `dcsync` alias which is a wrapper around Mimiktaz.  The adversary is free to pull any (or all) data from a DC, however a common target is the _krbtgt_ account.  Recall from the [Kerberos chapter](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops\&unit=674b79df7385fdbe1c0e7737) that the secret (password hash) for this account is used to encrypt and sign TGTs.  Therefore, once an adversary possesses this hash, they can forge 'legitimate' (or at least, valid) TGTs for any user in the domain.

Although there is a [manual process](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-reset-the-krbtgt-password) for changing the password for the krbtgt account, it is not done automatically by Active Directory.  This makes it a very good candidate for long-term, high-privileged access, to a domain.

### OPSEC <a href="#el_1738145783095_524" id="el_1738145783095_524"></a>

Because DRS is legitimately used, its mere presence does not constitute a breach in the environment.  Defenders must look for anomalous replication requests that stand out from the norm, for example those that originate from IPs other than known domain controllers.

When Directory Service Access auditing is enabled, these replications are logged as 4662 events.  The identifying GUID is `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` for **DS-Replication-Get-Changes** and **DS-Replication-Get-Changes-All**, or `89e95b76-444d-4c62-991a-0facbeda640c` for **DS-Replication-Get-Changes-In-Filtered-Set**.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/2156021a3b2a7938400b651c5024ee49.png" alt=""><figcaption></figcaption></figure>
