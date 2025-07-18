# Persistence

_Persistence_ is a tactic \[[TA0003](https://attack.mitre.org/tactics/TA0003/)] used to maintain access to a compromised system across reboots and other interruptions.  Techniques typically involve making configuration changes that will re-execute a payload on a pre-defined trigger or event.  Persistence is important, particularly for long-term access, because an adversary may not be able to reliably regain a foothold via their initial access technique.

Persistence may be added under the context of a standard user or local admin.  The latter allows an adversary to maintain high-privileged access to the system, but is only possible if they've been able to perform [privilege escalation](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops\&unit=674b76f01ed1d05747055009).  To avoid conflating techniques, this section will solely cover userland persistence techniques and elevated techniques are covered in [another chapter](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops\&unit=674b787d8578d242740985bc).

* Registry Run Keys & Startup Folder \[[T1547.001](https://attack.mitre.org/techniques/T1547/001/)].
* Logon Scripts \[[T1037.001](https://attack.mitre.org/techniques/T1037/001/)].
* PowerShell Profile \[[T1546.013](https://attack.mitre.org/techniques/T1546/013/)].
* Scheduled Tasks \[[T1053.005](https://attack.mitre.org/techniques/T1053/005/)].
* Component Object Model Hijacking \[[T1546.015](https://attack.mitre.org/techniques/T1546/015/)].
