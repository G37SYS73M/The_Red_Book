# Azure VMs - Azure AD Devices

### Azure VMs - Azure AD Devices

Three types of device identities

* Microsoft Entra join
  * Organization owned devices and heavily managed using Intune or Configuration Manager
  * Only Windows 11 and 10 and Server 2019 machines running on Azure
  * Can be accessed using Azure AD account
* Microsoft Entra registration
  * Can be user owned (BYOD) or organization owned. Lightly managed
  * Windows 10 or newer. macOS, Ubuntu and mobile devices
* Microsoft Entra Hybrid Join
  * Organization owned devices joined to on-prem AD and registered with Entra ID
  * All supported Windows Desktop and Server versions
* When a machine is joined to Entra ID, following users/roles are made a member of the local administrators group for management
  * Global Administrators
  * Microsoft Entra Joined Device Local Administrator
  * User who joined the machine to Azure
* Other Azure users can also be joined to local administrators group of Entra joined machines.
