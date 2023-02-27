# Roles & Privileged Identity Management
# Administrative Units

Administrative Units (AUs) can be considered much like AD's Organisational Units, being that they provide a container of resources to which you can delegate management of. This can be used to support complex resource and identity management structures, and so should be reviewed to check for AUs containing high-value users. The following roles can be used to manage users in administrative units:
-   Authentication administrator
-   Groups administrator
-   Helpdesk administrator
-   License administrator
-   Password administrator
-   User administrator
## AAD Roles and Assignment

As discussed, Azure AD is Microsoft's cloud-based identity and access management service in Azure and in general across their SaaS offerings as well. It has some similarities to on-premises Active Directory but also some significant differences.

Now in terms of access within Azure AD there isn't, in theory, a global admin that has access to all resources across all subscriptions *by default*. So, to get a better idea of how Azure manages access please refer to this diagram provided by Microsoft in their Azure documentation [located here](https://docs.microsoft.com/en-gb/azure/role-based-access-control/media/rbac-and-directory-admin-roles/rbac-admin-roles.png).

The two boxes that we care about are the green and the blue box. The green Azure AD roles (aka Administrative roles) refer to the level of access a user has to Azure AD specifically. What that means is that that level of access does not translate to the underlying projects and resources (VMs, storage, subnets, etc.) deployed in Azure.

* They are only meant to manage permissions for *identities* in AD itself
* They cannot be set to groups by default - if you really need to give, say, Security Admin privileges to users of a group, you need to assign them to each individual user
  * A new `isAssignable` property can be set when creating a group, which will allow it to be assigned an Azure AD role

The blue box refers to Azure (resource) roles (aka RBAC roles). As such it represents a high-level representation of access to Azure projects within a given tenant. This is where all the deployed resources and services would be found within a given subscription.

* They are meant for the resources and subscriptions containing these resources
* They can be assigned to both users and groups

Normally the Azure RBAC roles and Azure AD roles are separated to keep complete access away from one user at any given point in time. ***However***, a user with the Global Administrator role within Azure AD can elevate their privileges by enabling the "Access management for Azure resources" control in Azure AD > Properties. This would then assign them the "User Access Administrator" role at the root (/) level, which translates to *all* subscriptions in the tenant. With this, the GA can now assign themselves *or others* any level of access to Azure resources.

Here is a quick list of the most commonly encountered roles within Azure AD as well as the general level of permissions assigned to them. If you want to see specific details and permissions assigned to each you can check over [here](https://docs.microsoft.com/en-gb/azure/active-directory/users-groups-roles/directory-assign-admin-roles#role-permissions).

| Role | Level of access |
|:-----|:----------------|
|Global Administrator|User with full access to all administrative features in AAD as well as any services that use AAD (Microsoft365, Exchange Online, Dynamics365)|
|Global Reader| So global reader represents the read-only equivalent of Global Administrator, however, it is not exactly there yet atm. There are several limitations regarding the role (OneDrive admin center does not work, very limited access to M365 admin center, Office Security&Compliance Center, Teams Admin Center and some other bits). Overall, this would preferably be the role we would ideally like when reviewing environments, however the cases in which organisations have actually given us this role are rare. |
|Privileged Role Administrator|This is mostly related to Azure PIM (which is discussed in the following sub-section). This role would grant a user the ability to manage role assignments within AAD and Azure AD PIM. As such, this role can ultimately grant users the ability to become Global Administrators. |
|Directory Readers|Users can read basic directory information. Is usually used to grant a set of users read access to the Azure portal and general read-only information about a specific directory within the Azure environment|
|Security Reader|Users with this role have global read-only access to most security related features in Azure AD. Can see role assignments in Azure PIM as well. Usually what we request for assessments alongside Reader access to the relevant subscription where the resources lie. |

Now to compliment the list above, let us do a quick table of relevant roles within Azure. Again, detailed permissions can be seen in the following link from Microsoft's documentation [https://docs.microsoft.com/en-gb/azure/role-based-access-control/built-in-roles]

| Role | Level of access |
|:-----|:----------------|
|Owner|The Owner role provides full access to all Azure resources within the assigned scope of the user's role. This can be limited in a granular fashion to just a resource group or to all subscriptions in the tenant as well. |
|Contributor|The only difference between Owner and Contributor is that Contributor does not have permissions to perform any of the user access management tasks. As such, if the intent is to compromise machines or deploy new ones, you realistically would only need Contributor access|
|Reader|Allows a user full read-only access to resources within the assigned scope to your role. Again, can be pretty granular but usually on assessments we would like to get Reader access assigned at a subscription level ideally to see all relevant resources. |
|User Access Administrator|Lets you manage user access to Azure resources. Overall, not something you want to give to people often even at a relatively constrained scope as they would have full access to anything within that scope. |

Before moving on, now that we've got a better understanding of these basic concepts, there's a few more terms we need to introduce/explain better. It's usually valuable for services or apps to have identities that can allow them to directly perm actions such as grabbing secrets material without needing hardcoded credentials. To address these issues Azure AD provides two methods:

- **Service principals**
  - An *identity* is just a thing that can be authenticated (user with creds, app with keys/certs)
  - A *principal* is an identity acting with certain roles or claims, not really separate from identity. This can be imagined in a way similar to running "sudo" in a terminal: you're still the same user, but you changed the role under which you execute. Groups can be considered principals as they can have rights assigned to them.
  - A *service principal* is an identity that is used by a service or application. And like other identities, it can be assigned roles.
- **Managed identities for Azure services**
  - Creating service principals is a tedious process and they are difficult to maintain, Managed identities will do most of the work for you. MI can be created instantly for any Azure service supporting it. It's effectively creating an account on an organization's AD tenant
  - The Azure infrastructure will take care of authenticating the service and managing the account, to then be used as any other Azure AD account, with access to other Azure resources

To get what user roles have been assigned to your account once you are logged into the Azure CLI. You can run:

```bash
az role assignment list --all
```

This would tell you all the assignments that have been provided to your account at the various scopes at which they have been assigned. As by default without the "--all" argument, it would fetch only assignments scoped to the subscription level. Take note of the "scope" key in the resulting JSON file as it signifies at what level your account has been assigned that role. Although not as useful for pre-defined roles, you can get the permissions assigned to custom roles in the environment. You can get all the custom roles in the Azure environment by doing the following:

```bash
az role definition list --custom-role-only true
```

Usually worth checking for any custom roles that have wildcard (\*) assignments, as these could have some unintended consequences that were assigned just to avoid having to specifically specify which actions that role should have.

### Azure AD Privileged Identity Management (PIM)

Azure AD PIM is Microsoft's service to manage and monitor access to resources within an organization. Essentially it attempts to solve the issue of users being set privileged user roles and then being left to their own devices for longer than needed.

It provides users with an approval-based role activation that can have a time-based expiration set to it. This would ensure that any sensitive operation is performed within a given time-frame and if required additional access requests can be raised. This also has the side-effect of providing a clear audit history when users request access to a given resource or subscription in the environment.

There are two general concepts relevant to PIM that would be crucial to understand what user has what access rights:

* eligible - this is a role assignment to a user. They can perform an "activation" of that role when needed before being able to use it. They will need to perform *an* action as part of that activation (MFA check, business justification, or request approval, etc.). These settings are all decided when creating the role assignment in PIM.
* ***active*** - this means the user does not have to perform any actions before being able to use the role, i.e. it's *directly assigned* as active. This means that even if for instance the role would normally require the user to MFA, a user assigned an "active" role won't be required to MFA
* ***activated*** - this means that a user had an **eligible role assigned to them**, they requested it to be enabled, have performed the necessary actions, and it's now enabled. Slight, but important distinction from the "active" role above.

In addition, despite PIM intending to restrict access to user role permissions to a given timeframe, it still allows administrators to set so called "permanent" attributes to the roles. This effectively could mean a user permanently can be eligible or have a role permanently be "active", thus removing the time limit component, but still keeping an audit log of assignments.

Finally, there's also the "expire active" and "expire eligible" roles, which essentially refer to the user able to use, and respectively activate a role, only within a specified start/end date. The main point here being that now you've not only got a time limit, you've also got a specific window in which you'd ever be able to use/activate that role. When that windows is finished, even your "eligible" status is gone. This feature can be useful for guest users.

If you are provided access to Azure PIM and want to see what roles have been assigned to you, then you can find any roles that have been assigned or that have been made eligible for activation in the "Privileged Identity Management" service and in the "My roles" blade of the dashboard. There you would have two categories for the two types of available roles: "Azure AD roles" for Azure AD and "Azure resources" for Azure RBAC roles.

To see the Azure AD role assignments, you would have to have the "**Security Reader**" role assigned to you. However, you will not be able to see the assignments for Azure resource roles just using that role. You would need an Azure role assigned to your account such as "Reader" at minimum to see assignments.

### User roles and permissions

To start off with, one good thing to understand is whether the organisation is using Azure AD Privileged Identity Management (PIM) or they are not. Go to the Privileged Identity Management service and check out what roles are available to you or have been assigned to you. If you have not been assigned your current roles, then it is likely that they are not using it as role assignment should normally all be handled through PIM otherwise what would the point of it be.

If you have been assigned access sufficient access and PIM is included in the scope of the assessment, then the best thing to review would be the settings applied to some of the highly privileged Azure AD roles. Go to "Settings" and check any of the accounts that have the Modified attribute set to yes. Investigate what attributes have been enabled or disabled and whether it seems reasonable from a security standpoint. Especially regarding whether permanent active assignments can be set.

Otherwise, in terms of high-privileged account access, verify the number of Global Administrators that are setup in the environment. Microsoft recommends as a best practice to have between 2-4 accounts depending on how many backup accounts you would require. At minimum, all high-privileged accounts should have MFA enabled. Details on how to check and everything you would ever need to know about MFA in Azure [can be found here](/azure/services/azure_MFA).

Once that is established, it is worth checking that the domain does not have a vast amount of guest user accounts. Mainly as these accounts could potentially be overlooked so worth checking for the existing of them. However, if the organisation requires active collaboration with a large variety of organisations in a rapid fashion, then a better approach would be working with the IT team to setup some restrictions that would be applied to guests via Conditional Access Policies. Verifying the amount of guest users can be done by just querying the list of users by doing the following:

```bash
az ad user list --query "[?additionProperties.userType=='Guest']"
```

Following this, it is time to move to user roles and permissions. I would list all custom roles and assess what those roles allow a user to do. This is in an attempt to see if there are any overprivileged user roles in either scope or in the actions that they can do. List all roles using:

```bash
az role definition list --custom-role-only true
```

Here is an example of a high-privileged Azure role, albeit this is one of the predefined ones. Generally you can see the fact that the scope is assigned "/" which means the directory root and would provide you access to all subscriptions under it:

```json
{
    "assignableScopes": [
      "/"
    ],
    "description": "Lets you manage user access to Azure resources.",
    "id": "/subscriptions/af009c5c-27ca-43c9-8471-b26857d9ac87/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "name": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "permissions": [
      {
        "actions": [
          "*/read",
          "Microsoft.Authorization/*",
          "Microsoft.Support/*"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "User Access Administrator",
    "roleType": "BuiltInRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
```

Lastly, it is worth reviewing the actual Azure roles assigned to users in the environment. As kind of mentioned, Azure has 4 primitive roles which are "Owner", "Contributor", "Reader" and "User Access Administrator". Ultimately it is decently common for projects or resources to have an inordinate number of users with Owner or Contributor level access to a resource. As you might suspect, this does give them a lot of privileges and if they are Owners, they can assign access to other accounts. Good practice is to not have more than 3 at the subscription level, but obviously this might be a bit context dependant so worth discussing with the organisation you're reviewing if possible to figure out who has Owner/Contributor access and whether they would still need it.

