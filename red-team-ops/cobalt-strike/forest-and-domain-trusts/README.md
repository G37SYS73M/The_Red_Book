# Forest & Domain Trusts

Many Active Directory environments are more complex than a single, isolated, domain.  They have trust relationships with other forests and/or domains.  The purpose of a trust is to allow one forest or domain to share its resources with another.  Trusts can exist between domains in the same forest, between domains in different forests, and even between entire forests.  The following summarises the types of trust you're more likely to encounter:

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/1e68ec5811cb75b51d2c443ffa6ba69e.png" alt="" width="100%"><figcaption></figcaption></figure>

*   **Parent/Child Trust**

    A two-way, transitive trust that is automatically created when a new domain is added to an existing tree.
*   **Tree-Root Trust**

    A two-way, transitive trust that is automatically created when a new domain tree is added to an existing forest.
*   **External Trust**

    A one or two-way, non-transitive trust that enables resources to be shared between domains in different forests.
*   **Forest Trust**

    A one or two-way transitive trust that enables resources to be shared between different forests.

### Transitivity <a href="#el_1738582162233_558" id="el_1738582162233_558"></a>

The _transitivity_ of a trust determines whether the trust relationship should extend beyond the two parties with which it was formed.  Imagine a scenario where Domain A has an explicit trust relationship with Domain B; and Domain B has an explicit trust relationship with Domain C.  If these trusts were transitive, then Domain A would also implicitly trust Domain C.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/95aa151c7231d9cc09566d079d7d5f67.png" alt="" width="100%"><figcaption></figcaption></figure>

### Trust Direction <a href="#el_1738582600812_649" id="el_1738582600812_649"></a>

The direction of a trust enables access to resources in either one or both directions.  A one-way trust between Domain A and Domain B would allow users in Domain A to access resources in Domain B; but users in Domain B cannot access resources in Domain A.  A two-way trust obviously enables access to resources in both directions.

One-way trusts are also called **inbound** or **outbound** depending on which side of the trust you're on.  In the example below, the one-way trust is inbound from the perspective of Domain A; and outbound from the perspective of Domain B.  Some documentation refers to this as the **trusting** domain and **trusted** domain.  Confusingly, the direction of the trust is opposite to the direction of access.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/b8c4c297c03fcaa48983ac438a55a77c.png" alt=""><figcaption></figcaption></figure>

Two-way trusts do not actually exist in Active Directory, they are just two one-way trusts in opposite directions.

### Trusted Domain Objects <a href="#el_1738582922860_777" id="el_1738582922860_777"></a>

Information about each trust relationship is stored in Active Directory as a Trusted Domain Object (TDO).  This includes the trust type, transitivity, and the shared password used to create it.  The primary domain controller in the trusting domain changes the TDO password every 30 days and propagates it to a domain controller in the trusted domain.  TDO's can be read by querying the `trustedDomain` object class.

```
beacon> ldapsearch (objectClass=trustedDomain)
```

Important attributes include:

*   **cn**

    This is the the fully qualified domain name (FQDN) of the domain.
*   **flatName**

    This is the NetBIOS name of the domain.
*   **trustDirection**

    Specifies the direction of a trust, where:

    * **0** is `TRUST_DIRECTION_DISABLED`.
    * **1** is `TRUST_DIRECTION_INBOUND`.
    * **2** is `TRUST_DIRECTION_OUTBOUND`.
    * **3** is `TRUST_DIRECTION_BIDIRECTIONAL`.
*   **trustAttributes**

    These are a set of bitwise flags that define various properties of the trust.  I shan't list them all here, but relevant ones include:
* **1** is `TRUST_ATTRIBUTE_NON_TRANSITIVE`.
* **4** is `TRUST_ATTRIBUTE_QUARANTINED_DOMAIN` and means SID filtering is in place.
* **8** is `TRUST_ATTRIBUTE_FOREST_TRANSITIVE` and means the trust is transitive between two forests.
* **32** is `TRUST_ATTRIBUTE_WITHIN_FOREST` and means the trust is between two domains in the same forest.
* **64** is `TRUST_ATTRIBUTE_TREAT_AS_EXTERNAL` and means that the trust is between two domains in different forests.  SID filtering is also implied.

### Security Boundaries <a href="#el_1738582905645_750" id="el_1738582905645_750"></a>

In this context, a 'security boundary' defines the scope of authority for administrators in these forests and domains.  It may be obvious to say that an enterprise administrator has authority over their entire forest, and a domain administrator only has authority over their particular domain.  It would therefore be logical to assume that a domain administrator should not be able to administer other domains in their forest, otherwise a security boundary would have been broken.  However, this is not the case.

Officially, a security boundary only exists at the forest level.  It is not possible to prevent administrators from one domain accessing data in another domain, if they are part of the same forest.  This will become particularly evident when we look at how an adversary can hop from a child domain, up to the tree-root.
