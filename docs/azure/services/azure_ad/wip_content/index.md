# Azure Active Directory (AAD)

## Glossary of terms

Before we get started here is some key terminology that will be useful when trying to wrap your head around some of the AAD concepts.

| Terminology | Definition |
|:------------|:-----------|
| Tenant | Effectively the base unit that represents an organization in Azure. It is an instance of Azure AD that is created when an organization signs up for cloud services with Microsoft. It is used to manage user permissions and access to not only Azure services but to other Microsoft SaaS offerings as well (Microsoft365, Intune, Dynamics365). |
| Subscriptions | Effectively a separate payment agreement between the organization in question and Microsoft within their Azure tenant. You can have as many subscriptions as you want, they would logically separate your resources based on how you want to manage their costs. This would be a common high-level point where organisation resources would be segregated at. |
| Identity | An entity that can have permissions assigned to it and authenticate to the platform. Users can be thought of having an identity because they'd authenticate with creds, same as an app authenticating with certificates or keys, can therefore be thought of having an identity. |
| Azure AD account | An identity created through Azure AD which is stored in AAD and accessible to all the organization's cloud service subscriptions|
| Azure AD directory | Each tenant has a dedicated Azure AD directory. It is made up of all the identities associated to the organization (users, groups, and apps) and is used to perform identity and access management functions|
|Azure Management group|A high-level abstraction of a group of several subscriptions. This is used to centralise management of access, policies and compliance related to several relevant subscriptions. All subscriptions in a management group inherit conditions assigned to the group.|

## Assessment notes

### Verify access

As a start, check if you have access to any subscription by going to the "Subscriptions" service in Azure. A good place to start as you would usually be assigned access to the relevant subscription where the resources, you are going to test are hosted in. If there is nothing there, then it is likely you don't have any assigned access, but it might not be necessarily true. You can further verify this by going to "All resources" in the search bar and if nothing shows up there then you likely have an Azure AD user that is not assigned any Azure roles.

You can further verify what access you have by doing:

```bash
az role assignment list --all
```

If you are integrating with a Microsoft SaaS product, then you won't necessarily need access to Azure resources to access those solutions via Azure AD. Here are some common Microsoft SaaS offerings that you might actually have access to:

- dev.azure.com - Azure DevOps (could be connected to Azure AD or separately managed with Microsoft accounts)
- admin.microsoft.com - Microsoft Admin panel
- endpoint.microsoft.com - Microsoft Device Management Endpoint
- home.dynamics.com - Dynamics365 panel

### AZ CLI and PowerShell cmdlets

There are three main ways to interact with Azure programmatically apart from the web dashboard. This is done through the Azure CLI, Azure PowerShell cmdlets or the Azure REST API. Links to how to install and work with the three components can be seen below:

* [AZ CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* [AZ PowerShell](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-4.5.0)
* [AZ REST API](https://docs.microsoft.com/en-us/rest/api/azure/)

Additionally, you should be aware that interacting with Azure Active Directory is done via a different set of PowerShell modules. Specifically it would be the following two, the first of which is the main one to use and the second is the older version which will be eventually deprecated:

* [AzureAD PowerShell](https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-adv2?view=azureadps-2.0)
* [MSOL PowerShell](https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-msonlinev1?view=azureadps-1.0)
* [Microsoft Graph SDK for PowerShell](https://docs.microsoft.com/en-us/powershell/microsoftgraph/overview?view=graph-powershell-1.0)

One thing to keep in mind with the CLI and PowerShell modules is that commands that you run will by default be run against what is configured as a **default subscription**. A default subscription is assigned after you have logged in and created a valid access token for the CLI or PowerShell. Checking what is the current configured default can be done using:

```bash
az account show # AZ CLI command to show the current default command
```

```PowerShell
Get-AzContext # AZ PowerShell command to get the current subscription set as the default
```

Now this is not an issue when you only have access to one subscription, but it is often that you might have several subscriptions in which case **running commands will only query the default subscription and won't show you stuff in the other ones**.

Best way to approach running tests and verifying issues when you're working on several subscriptions is to use just some simple for loops in either bash or PowerShell to iterate through all relevant subscriptions as you can always specify which subscription will the command be run against. This is done by using the "--subscription "<sub_name>" parameter. A simple example loop to fetch all the Azure Key Vaults in each subscription you have access it is the following using PowerShell and the Azure CLI:

```PowerShell
az account list --all --query "[].name" -o tsv
foreach ($sub in $subscriptions) {echo $sub; az keyvault list --subscription $sub}
```

```bash
az account list --all --query "[].name" -o tsv
while read sub; 
do 
  echo "$sub"; 
  az keyvault list --subscription $sub; 
done
```

If you do not want to have to add the "--subscription" parameter always, you also have the option of setting a default subscription or "current context" to your CLI or PowerShell session. In the Az PowerShell module you can set an explicit subscription and even Azure AD Tenant by running the following command:

```PowerShell
Set-AzContext -Subscription "xxxx-xxxx-xxxx-xxxx" -Tenant "xxxx-xxxx-xxxx-xxxx"
```

This is the equivalent one for the Azure CLI, note that for the Azure CLI you need to configure the organisation's tenant at the start if you've been provided access via a guest account using your F-Secure email. So make sure you do this, otherwise it might default to running commands against whatever home(default) tenant your account is assigned to. The value of the "--tenant" parameter could either be the ".onmicrosoft.com" or custom domain for that organisation or the Azure object ID (easiest way to find is after logging in via the browser):

```bash
az login --tenant "xxxx-xxxx-xxxx-xxxx"
az account set --subscription XXX
```

On a similar note, it is important to point out how to authenticate to the Azure AD PowerShell module via a guest account access. When you've been given access via your main account, you should make sure that you authenticate correctly to their Azure AD tenant as otherwise all the commands will attempt to execute to your home organisation. When connecting to the AzureAD PowerShell module you can do the following:

```PowerShell
Connect-AzureAD -AccountId $AdminUPN -TenantId "xxxx-xxxx-xxxx-xxxx"
```

### Removing access following engagement

Credentials and access via the CLI or PowerShell modules could persist for longer then you could expect and if not cleared up, you might end up running commands against infrastructure that you don't own. As such, it is important to clear up your active sessions after each engagement or review. Here is how to clear your shell/terminals from the various CLI or PowerShell modules discussed above:

* Azure CLI

```bash
az logout
```

* Az PowerShell

```PowerShell
Disconnect-AzAccount -Scope CurrentUser
Clear-AzContext -Scope CurrentUser
Clear-AzDefault -Scope CurrentUser
```

* AzureAD Powershell

```PowerShell
Disconnect-AzureAD
```

This should complete all the access removal if you have been provided an account in their organisation tenant for the duration of the assessment.

In the event you have been granted access via a guest user account and you used your main account, then there is one extra step to perform. Go to [https://myaccount.microsoft.com/organizations](https://myaccount.microsoft.com/organizations) and once you've logged in with your credentials you should see a list of the organisations you are a member of at the moment. Once you've finished a review of a cloud environment there is no need for you to stay a member of their organisation and as such, you should click "Leave organization" on any organization you are a member of.

#### Microsoft Defender for Cloud

Centralised portal where you can get alerts and recommendations on what actions are needed to improve the organization's security score. Really useful as a sort of starting point as it usually should pick up all sorts of low-hanging fruit. Just be sensible with recommendations especially around some low risk issues.

An interesting feature of Defender for Cloud is that you get *alerts* as we said, but it will also correlate data and create *incidents*, in which you can also see the set of alerts that are related to that individual incident, and gives analysts a better view into what happened and the set of actions that caused what.

One thing to keep in mind is that it can sometimes mistakenly complain about MFA not being setup, but that's only if they have the premium edition license for Azure AD. Azure AD free edition can still enforce MFA requirement for all users by using [Security defaults](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/concept-fundamentals-security-defaults), but Security Center will still complain that MFA is not enabled. So as mentioned before, for MFA as we will rarely get proper access to figure out whether it's implemented, worth just asking and discussing with an organisation. Usually checking MFA status would require access to: (account.activedirectory.windowsazure.com)


