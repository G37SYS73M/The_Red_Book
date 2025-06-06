# Hybrid Identity

* Organizations have resources, devices and applications both onpremises and in the cloud.
* Many enterprises use their on-prem AD identities to access Azure applications to avoid managing separate identities on both.
* "A single user identity for authentication and authorization to all resources, regardless of location…is hybrid identity."
* An on-premises AD can be integrated with Entra ID using Entra Connect with the following methods. Every method supports Single Sign-on (SSO):
  * Password Hash Sync (PHS)
  * Pass-Through Authentication (PTA)
  * Federation
* For each method, at least the user synchronization is done and an account MSOL\_\<installationidentifier> is created on the on-prem AD.

#### Hybrid Identity - Cloud Sync

* Cloud Sync can also be used for Hybrid Identity.
* It uses a 'lightweight' agent in place of the Entra Connect application.
* Cloud Sync doesn't support PTA.

#### Hybrid Identity - PHS

* It synchronizes users and a hash of their password hashes (not clear-text or original hashes) from on-prem AD to Entra ID.
* The simplest and most popular method for getting a hybrid identity.
* PHS is required for features like Identity Protection and AAD Domain Services.
* Hash synchronization takes place every two minutes.
* When a user tries to access any Azure resource, the authentication takes place on Entra ID.
* Built-in security groups are not synced.
* By default, password expiry and account expiry are not reflected in Entra ID. That means a user whose on-prem password is expired (not changed) can continue to access Azure resources using the old password.
