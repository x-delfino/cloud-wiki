import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Guests & External Identities

Now one thing to clarify off the bat is that, whilst they're often used iterchangeably, **Guests** and **External Identities** are not exactly synonymous terms.
- **Guests**: accounts with a `UserType` of `Guest` as opposed to `Member` which has an impact on their level of access to resources (we'll go into this later). 
- **External Identities**: accounts that are homed in an external identity provider such as a user from an external AAD Tenant or with a Microsoft account. 

External collaborators are added to AAD as guests by default, but they can be changed; likewise accounts within the local AAD tenant can be set as guest accounts. Make sense? It's a small distinguishment but does help with understanding how they work. 

## Guests

Broadly, guest users can be broken into one of the five following types:
- A user exists in an **external AAD tenant** and is invited to the inviting organisation's AAD tenant as a guest. The guest user uses their external AAD credentials to access resources in the inviting organisation.
- A user exists as a **Microsoft account** and is invited to the inviting organisation's AAD tenant as a guest. The guest user uses their Microsoft account credentials to access resources in the inviting organisation.
- A user exists as a **non-Microsoft/non-AAD account** and is invited to the inviting organisation's AAD tenant as a guest. The guest user verifies their email address using an OTP to access resources in the inviting organisation. This process is called just in time (JIT) tenancy.
- A user created in the **inviting organisations on-premise AD** and synced up to AAD. The inviting organisation manages the guest users credentials.
- A user created in the **inviting organisations AAD**. The inviting organisation manages the guest users credentials.

Guest users can identified by the `userType` property value of `Guest` and can be retrieved using the following methods:
<Tabs>
  <TabItem value="posh_aad" label="AzureAD PowerShell" default>

```powershell title="Query"
Get-AzureADUser -Filter "userType eq 'Guest'"
```

```text title="Response"
ObjectId                             DisplayName UserPrincipalName                                   UserType
--------                             ----------- -----------------                                   --------
eb4babef-9b09-xxxx-xxxx-xxxxxxxxxxxx 12345, user user.12345_company1.com#EXT#@3hbxxx.onmicrosoft.com Guest
```

  </TabItem>
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell title="Query"
Get-MgUser -Filter "userType eq 'Guest'"
```

```text title="Response"
Id                                   DisplayName Mail                    UserPrincipalName                                   UserType
--                                   ----------- ----                    -----------------                                   --------
eb4babef-9b09-xxxx-xxxx-xxxxxxxxxxxx 12345, user user.12345@company1.com user.12345_company1.com#EXT#@3hbxxx.onmicrosoft.com
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/v1.0/users?$filter=userType+eq+'Guest'
```

  </TabItem>
  <TabItem value="posh_az" label="Az PowerShell">

```powershell title="Query"
Get-AzAdUser -Filter "userType eq 'Guest'"
```

```text title="Response"
DisplayName Id                                   Mail                    UserPrincipalName
----------- --                                   ----                    -----------------
12345, user eb4babef-9b09-xxxx-xxxx-xxxxxxxxxxxx user.12345@company1.com user.12345_company1.com#EXT#@3hbxxx.onmicrosoft.com
```

  </TabItem>
  <TabItem value="az" label="Azure CLI">

```bash title="Query"
az ad user list --filter "userType eq 'Guest'" -o table
```

```text title="Response"
DisplayName    GivenName    Mail                     UserPrincipalName
-------------  -----------  -----------------------  ---------------------------------------------------
12345, user    user         user.12345@company1.com  user.12345_company1.com#EXT#@3hbxxx.onmicrosoft.com
```

  </TabItem>
</Tabs>

### Access Level

The thing that really distinguishes guests is their level of access. This can be set to one of the following options:

| Permission Level         | Setting                                                                                                         | ID                                   |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Same as member users     | Guest users have the same access as members (most inclusive)                                                    | a0b1b346-4d3e-4e8b-98f8-753987be4970 |
| Limited access (default) | Guest users have limited access to properties and memberships of directory objects                              | 10dae51f-b6af-4016-8d66-8c2a99b929b3 |
| Restricted access        | Guest user access is restricted to properties and memberships of their own directory objects (most restrictive) | 2af84b1e-32c8-42b7-82bc-daa82404023b |

This can be checked using the following methods. They'll return the IDs which you can match against the above table.

<Tabs>
  <TabItem value="posh_aadpreview" label="AzureADPreview PowerShell">

```powershell title="Query"
(Get-AzureADMSAuthorizationPolicy).GuestUserRoleId
```

```text title="Response"
10dae51f-b6af-4016-8d66-8c2a99b929b3
```

  </TabItem>
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell title="Query"
(Get-MgPolicyAuthorizationPolicy).GuestUserRoleId
```

```text title="Response"
10dae51f-b6af-4016-8d66-8c2a99b929b3
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/v1.0/policies/authorizationPolicy
```

  </TabItem>
</Tabs>

## External Identities

There are a couple of types of external identity access in AAD:
- **B2B Collaboration:** This is the most common form of external access. External collaborators are added as guests to the local AAD tenant to access resources. This means that they can be added to groups and assigned roles etc.
- **B2B Direct Connect:** This method requires a trust relationship be established with an external AAD organisation. External collaborators are **not** added to the local AAD tenant but are instead managed in the external tenant. Trust controls can determine whether MFA, device compliance and hybrid join status from the external tenant are trusted by conditonal access policies.

There's a full breakdown of use-case differences [here](https://learn.microsoft.com/en-us/azure/active-directory/external-identities/external-identities-overview#comparing-external-identities-feature-sets). In practicality, Direct Connect only has particular benefits in shared Teams channels at the moment and doesn't support the user management functionality of Collaboration.

### B2B Collaboration

B2B collaboration supports a range of identity providers (IdP) which handle authentication for external collaborators:

| Provider                          | Enabled by Default | Description                                                                                                                                                 |
| --------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AAD Accounts**                  | Yes                | External users use their AAD work or school accounts to access B2B resources.                                                                               |
| **Microsoft Accounts**            | Yes                | External users use their personal Microsoft accounts to access B2B resources.                                                                               |
| **Email one-time passcode (OTP)** | No                 | External users verify their email address using an OTP to access resources in the inviting organisation. This process is called just in time (JIT) tenancy. |
| **Google**                        | No                 | External users use their Gmail accounts to access B2B resources.<br/>Can be used for self-service signups and invited guests.                               |
| **Facebook**                      | No                 | External users use their facebook accounts to access B2B resources.<br/>Can only be used for self-service signups.                                         |
| **SAML/WS-Fed**                   | No                 | Configure federation with any external IdP that supports SAML or WS-Fed.<br/>Can only be used for invited guests.                                           | 

External collaborators can be easily identified in AAD by their user principal name format of:
`<username>_<external_domain>#EXT#@<aad_domain>`
eg. `name_withsecure.com#EXT#@aadtenant.onmicrosoft.com`

Now, how do these accounts actually get into the AAD? There's a couple of ways:
- **Invites:** Invite external collaborators as guests, either individually or in bulk.
- **Self-service sign-up:** Allow self-service signup to grant external collaborators access to specific resources.

#### Guest Invites

Admins can set the invite policy at the tenant level to configure which users can invite external collaborators as guests. The following options are available:
- Anyone in the organization can invite guest users including guests and non-admins (most inclusive)
- Member users and users assigned to specific admin roles can invite guest users including guests with member permissions
- Only users assigned to specific admin roles can invite guest users
- No one in the organization can invite guest users including admins (most restrictive)

The invite policy can be checked using the following methods:
<Tabs>
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell title="Query"
(Get-MgPolicyAuthorizationPolicy).AllowInvitesFrom
```

```text title="Response"
everyone
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/v1.0/policies/authorizationPolicy
```

  </TabItem>
</Tabs>

Restrictions on invitations can be applied based on an allow-list or deny-list of domains, defaulting to allowing collaboration with any domain. An allow-list approach is most restrictive and most secure here.

They can be retrieved using the following method (this looks to currently only be available on the Azure AD graph, not the Microsoft Graph):

<Tabs>
  <TabItem value="posh_aadpreview" label="AzureADPreview PowerShell">

```powershell
((Get-AzureADPolicy -All $true | ?{$_.Type -eq 'B2BManagementPolicy'}).Definition | ConvertFrom-Json).B2BManagementPolicy.InvitationsAllowedAndBlockedDomainsPolicy
```

  </TabItem>
</Tabs>

#### Self Service Sign-up

[User Flows](https://learn.microsoft.com/en-us/azure/active-directory/external-identities/self-service-sign-up-user-flow) can be used to support self service signup by allowing external users to access specific resources without requiring an invite.
Self service signups can be disabled at the tenant level; you can check whether it is permitted using the following methods:
<Tabs>
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell
(Get-MgPolicyAuthenticationFlowPolicy).SelfServiceSignUp
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/policies/authenticationFlowsPolicy
```

  </TabItem>
</Tabs>

The user flows themselves can be retrieved using the following methods:
<Tabs groupId="az-tool">
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell
Get-MgIdentityB2XUserFlow
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/identity/b2xUserFlows
```

  </TabItem>
</Tabs>

### Cross-tenant Access settings

Cross-tenant access settings can be used to provide granular control with external AAD organisations, both from an inbound and outbound perspective. By default, B2B collaboration (as outlined above) is enabled and B2B direct connect is disabled. The following policies can be configured:

| Setting            | Scope                                                                  | Inbound/Outbound | Description                                                                                                                                     |
| ------------------ | ---------------------------------------------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| B2B Collaboration  | Users <br/> Groups <br/> Applications                                  | Both             | Allow all or specified users, groups or applications to access external resources. They will be added as guests to the inviting tenant.         |
| B2B Direct Connect | Users <br/> Groups <br/> Applications                                  | Both             | Allow all or specified users, groups or applications to access external resources. They will **not** be added as guests to the inviting tenant. | 
| Trust Settings     | Trust MFA <br/> Trust Device Compliance <br/> Trust Hybrid Join status | Inbound          | Determine whether to trust authentication controls as verified by the external users home AAD tenant (relevant for Conditional Access).         |

These can be set as a default configuration, as well as on a per-organisation basis.

The default configuration can be checked using the following methods:
<Tabs >
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell
Get-MgPolicyCrossTenantAccessPolicyDefault | ConvertTo-Json -Depth 5
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/v1.0/policies/crossTenantAccessPolicy/default
```

  </TabItem>
</Tabs>

And each of the organisational specific configurations can be retrieved using:
<Tabs>
  <TabItem value="posh_graph" label="Microsoft.Graph PowerShell">

```powershell
Get-MgPolicyCrossTenantAccessPolicyPartner | ConvertTo-Json -Depth 4
```

  </TabItem>
  <TabItem value="graph" label="Graph API">

```http
GET https://graph.microsoft.com/v1.0/policies/crossTenantAccessPolicy/partners
```

  </TabItem>
</Tabs>