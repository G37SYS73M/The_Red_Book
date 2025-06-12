# Initial Access

### Phishing Taxonomy <a href="#el_1740649758449_429" id="el_1740649758449_429"></a>

[Mariusz Banach](https://x.com/mariuszbit) proposes a taxonomy for phishing payloads based on real-world observations of adversary behaviour.  He represents it as `DELIVERY(CONTAINER(TRIGGER + PAYLOAD + DECOY))` where:

* **Delivery** is the technique used to deliver the package to the victim.
* **Container** is the container format used to package the files.
* **Trigger** is the means to trigger payload execution.
* **Payload** is the malicious code to execute.
* **Decoy** is a file to display to the victim.

When DFIR vendors complete their investigations into breaches, they often publish their findings related to the adversary's initial access, command & control, and post-exploitation techniques.  The following diagram is a mind map (produced by CloudSEK) of a campaign responsible for distributing Lumma Stealer malware.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/91a034087e934535dd6ade1e067ce8a7.png" alt="" width="100%"><figcaption></figcaption></figure>

The threat actors tricked victims into clicking a Windows shortcut (.lnk) file that was disguised as a PDF.  The LNK files were weaponised to download additional malicious content using mshta, which in turn downloaded obfuscated JavaScript.  This JavaScript executed PowerShell to download and unpack the malware, hide it on the compromised systems, and run it.  The complexity of the chain is primarily to make it more difficult for security solutions to detect automatically and for manual analysts to figure out what's going on.
