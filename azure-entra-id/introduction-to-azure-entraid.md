# Introduction To Azure EntraID

## Introduction To Azure EntraID

Azure Active Directory (Azure AD or AAD) now renamed to Entra ID is “Microsoft’s cloud-based identity and access management service.”

Azure AD can be used to access both:

* External resources like Azure Portal, Office 365 and
* Internal resources like on-premises applications.

**Entra ID - Some Terminology:**

* Tenant - An instance of Entra ID and represents a single organization.
* Entra ID Directory - Each tenant has a dedicated Directory. This is used to perform identity and access management functions for resources.
* Subscriptions - It is used to pay for services. There can be multiple subscriptions in a Directory.
* Core Domain - The initial domain name \<tenant>.onmicrosoft.com is the core domain. It is possible to define custom domain names too.

> Entra ID is not Azure. Entra ID is a product offering within Azure. Azure is Microsoft's cloud platform whereas Entra ID is enterprise identity service in Azure.

#### Azure Architecture <a href="#azure-architecture" id="azure-architecture"></a>

**Management Groups**

* **Management groups manage multiple subscriptions**, and all **subscriptions inherit** the conditions applied to the management group.
* **Subscriptions** within a management group belong to the **same Azure tenant**.
* **Management groups can be nested** under **other management groups**, creating a hierarchy.
* The **Root management group** is the top-level management group for each Azure directory.
* **Global administrators** can elevate their privileges to the **Root management group** when necessary.

**Subscriptions**

* An Azure **subscription is a logical unit** of Azure services **linked to an Azure account**.
* It **serves as a billing** and/or **access control boundary** within an **Entra ID Directory**.
* A **single Entra ID Directory** can have **multiple subscriptions**, but **each subscription can only trust one directory**.
* Any **Azure role assigned at the subscription level** **applies to all resources** **within that subscription.**

**Resource Groups and Resources**

* A **resource is a deployable item in Azure**, such as virtual machines (VMs), app services, or storage accounts.
* A **resource group acts as a container** to **organize resources**.
* **All resources** in Azure **must belong** to a **single resource group**.
* **Deleting a resource group deletes all the resources inside it.**
* Each **resource group has its own Identity and Access Management (IAM) settings**, and **any role applied to the resource group will apply to all resources within it**.

**Managed Identity**

* Azure allows **Managed Identities** to be assigned to resources such as app services, function apps, and virtual machines.
* **Managed Identity** uses **Entra ID tokens** to authenticate and access other resources like key vaults and storage accounts.
* It functions as a special type of **service principal** that is integrated with Azure resources.
* Managed Identities can be:
  * **System-assigned**: Tied to a specific resource and cannot be shared.
  * **User-assigned**: Has an independent lifecycle and can be shared across multiple resources.
* **Entra ID** focuses on managing **identities and access control** for users, apps, and services, while **Azure Resource Manager** focuses on the **deployment and management of Azure resources**.
* **Entra ID** governs who can access Azure services, whereas **ARM** governs **how** those services are created, managed, and structured.

In summary, **Entra ID** is about identity and access, while **ARM** is about managing the Azure resources those identities interact with.

**Entra ID vs On-Prem AD**

* The only similarity between **Entra ID** and **Azure Active Directory Domain Services** is that both provide **identity and access management** solutions.
* Although they may share similar terms, it's important to avoid viewing **Entra ID** through the lens of on-premises Active Directory (AD) concepts.
* **Entra ID** is **not** a cloud-based directory service; that functionality is provided by **Azure Active Directory Domain Services (AADDS)**, which offers a "domain controller as a service" experience.
* It is possible to integrate on-premises **AD** with **Entra ID** for a **hybrid identity** solution, enabling seamless authentication and access across cloud and on-prem environments.

**(Azure Role-Based Access Control) RBAC Roles**

* **Azure RBAC Roles** (or simply **Azure roles**) provides access management for Azure resources using the **authorization system of ARM**.
* There are over more than 120 built-in roles (473 as per [https://azure.permissions.cloud/builtinroles](https://azure.permissions.cloud/builtinroles)) and we can define custom roles too.

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252F88EBdkru1JnXdLYGcTJA%252Fimage.png%3Falt%3Dmedia%26token%3Dd777923a-5a15-4fea-b66c-51c77919a5db&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=a3076375&#x26;sv=2" alt=""><figcaption></figcaption></figure>

**(Azure Role-Based Access Control) RBAC Assignments**

> **Azure AD Object/Principal** HAS **Role** ON **Scope**.

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252Fe9GE0jIifHf3Li3HQqj9%252Fimage.png%3Falt%3Dmedia%26token%3D0256a6dc-aaff-4bf2-8a2b-dee1c57dd39a&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=93cf4e2d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

* **Security Principal:** An entity in **Entra ID**, which could be a **user**, **group**, **service principal**, or **managed identity** that requires access to resources.
* **Role Definition:** A set of **permissions** that defines what actions a security principal can perform (or be denied from performing), such as read, write, or delete.
* **Scope:** The **resource** to which the role is applied, covering different levels of hierarchy such as **Management Group -> Subscription -> Resource Group -> Resource**. This defines where the role permissions take effect.

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FlMyvqq7ShEqPYO1qAZe3%252Fimage.png%3Falt%3Dmedia%26token%3D25365560-49b1-4c83-8e93-90b561642f3d&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=30511ba8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

**Azure Attribute Based Access Control (ABAC)**

> We can think of this as RBAC with conditional attributes.

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252F0nWXRKCJ4vlW0XPtGsRn%252Fimage.png%3Falt%3Dmedia%26token%3D7dc28809-fa33-4422-bef5-d0c9ab7e4453&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=8b6a2ba3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FbRnwRlAzfMFLpKmIGLYR%252Fimage.png%3Falt%3Dmedia%26token%3Dea4f5ab4-abfa-4f0e-90e7-100481e40508&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=27c25220&#x26;sv=2" alt=""><figcaption></figcaption></figure>

**Key Differences Between Azure RBAC and Entra ID Roles:**

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**Common Entra ID Roles:**

1. Global Administrator
2. User Administrator
3. Security Administrator
4. Privileged Role Administrator
5. Application Administrator

**Common Azure RBAC Roles:**

1. Owner
2. Contributor
3. Reader
4. User Access Administrator
5. Virtual Machine Contributor

#### Entra - Editions <a href="#entra-editions" id="entra-editions"></a>

<figure><img src="https://g37sys73m.gitbook.io/~gitbook/image?url=https%3A%2F%2F1846927083-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-Mj9GKP_mf1AnaApJEIA%252Fuploads%252FEY2S23hzm4DEhmHq2n9B%252Fimage.png%3Falt%3Dmedia%26token%3D2ec58735-82c5-4613-b4ef-8e4869cc0430&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=f87ef74d&#x26;sv=2" alt=""><figcaption></figcaption></figure>
