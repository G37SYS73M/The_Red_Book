# Inter-Realm Tickets

across different realms.  If you need a refresher on how Kerberos works in the same realm, go back to the [Kerberos introduction.](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops\&unit=674b79df7385fdbe1c0e7737)  Essentially, a principal obtains a TGT from the KDC in their realm and uses it to request service tickets for services in that same realm.  However, the exact same process is not possible when requesting service tickets from a different realm.  Typical TGTs issued in a trusted realm cannot be decrypted by a trusting realm because it doesn't have access to the same krbtgt secret.  For this reason, an inter-realm key is required to bridge this cryptographic gap.

Fun fact - all parent/child trusts in a forest use the same inter-realm key, which is actually how trust transitivity is implemented.  Transitive trusts share the same inter-realm key, whereas unique keys are created for each new non-transitive trust.

When a client wishes to access a service in a trusting realm, it will send a TGS-REQ to its own KDC first.  This request includes a copy of their normal TGT from the trusted realm, the `realm` field is also set to that of the trusted realm, and the `sname` field contains the SPN of the target service.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/1567a745828933ec174bc3ab39410b29.png" alt="" height="400" width="631"><figcaption></figcaption></figure>

Upon receipt, the KDC will spot that the requested service is in a different realm; so instead of returning a TGS-REP containing a service ticket for the target service (which it can't do because it doesn't know the foreign SPNs secret); it returns a TGS-REP containing an inter-realm TGT.  This is simply a TGT that has been issued by the trusted realm, but is encrypted using the shared inter-realm key instead of the krbtgt secret.

Microsoft describes this as a 'TGS referral', so you may find resources that describe an inter-realm ticket as a 'referral' ticket.

The TGT that comes back has its realm field set to the trusted realm, but the SPN is set to the krbtgt service of the trusting realm.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/dbc49a9ef74bff26556d9a77e1609225.png" alt="" width="100%"><figcaption></figcaption></figure>

The client then uses this inter-realm TGT to send another TGS-REQ, this time directly to the KDC of the trusting realm.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/c1a5029ef3dcc4368d9aac3fa0ea5317.png" alt="" height="400" width="580"><figcaption></figcaption></figure>

The foreign KDC will decrypt the inter-realm TGT using its copy of the inter-realm key and returns the service ticket in a TGS-REP.  A high-level summary of this flow is as follows:

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/6e2d2d2de4a3668a1e8fea99314d9667.png" alt="" width="100%"><figcaption></figcaption></figure>

### Trust accounts <a href="#el_1742216642967_923" id="el_1742216642967_923"></a>

After a trust has been created and the inter-realm key shared, the ticket-granting service of the trusting realm is registered as a principal with the trusted realm's KDC.  They are typically registered using the flat name (i.e. NetBIOS name) of the opposing realm.  In this example, a principal called `PARTNER$` will be registered in the **CN=Users** container in _contoso.com._

You won't see them in tools like ADUC, but they can be found by querying for accounts with a samAccountType of `SAM_TRUST_ACCOUNT`.

```batch
beacon> ldapsearch (samAccountType=805306370) --attributes samAccountName

Binding to 10.10.120.1

[*] Distinguished name: DC=contoso,DC=com
[*] targeting DC: \\lon-dc-1.contoso.com
[*] Filter: (samAccountType=805306370)
[*] Scope of search value: 3
[*] Returning specific attribute(s): samAccountName

--------------------
sAMAccountName: PARTNER$
```

The inter-realm key is used as the password for this account and it's where the trusted domain fetches the shared key when needing to encrypt an inter-realm TGT. This is different to the trusting domain, where the shared key is stored inside the TDO. I'm sure this is a perfectly logical design choice by Microsoft...
