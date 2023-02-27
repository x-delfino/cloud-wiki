# Enterprise applications

A couple of things that can be configured improperly are part of the "User settings" configuration in AAD. This is found on the sidebar "User settings". These controls would be mainly related to the usage of enterprise applications that use your AAD as an identity system. These settings will not necessarily be wrong, but it is important to raise a discussion with an IT team on what business case exists for enterprise applications.

There are four general types of enterprise applications in Azure AD:

* Azure AD Gallery applications - Applications that are prepared to integrate with Azure AD. A lot of SaaS applications with native support fall in this category. [Here](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tutorial-list) is a list of them.
* On-prem applications with Azure AD Application Proxy - integrated on-prem web apps using Application Proxy to support single sign-on (SSO). Some more detail on the [Application Proxy](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/application-proxy) is available in their documentation.
* Custom-developed apps - apps that are registered with Azure AD to support single sign-on
* Non-Gallery apps - applications that support SSO using SAML or OpenID connect

If they would only expect certain users to on-board new applications that could use password single-sign on or should general users be able to perform so. Admittedly, I would argue that there should not be too many circumstances where these settings would be enabled to allow users to allow applications to access their data without being pre-vetted by administrators. But still, worth asking if unsure.

One thing to keep note of is that a lot of times, organisations use a lot of applications that use Azure AD as an identity provider without even having been approved apps by administrators. There was an excellent talk by Mark and Michael from Microsoft on enterprise application phishing which was presented at a BSides event which can be seen here:

- https://www.youtube.com/watch?v=mxOHcqHxpi8

Some important notes from the talk can be seen below alongside with some stats:

* 140 apps are used on average within an organisation
* 80% of employees use non-approved apps for work
* 87% of users can consent to applications

Ultimately, it's best to think about this in the same way as you think about mobile apps. When you install an app you often need to approve the permissions it will require for it's day-to-day operations. In the context of Azure AD, this would be the organisation data that an application can access through resource APIs that expose permissions. Ultimately, these permissions to access organisational data can be granted through application consent done by either an admin, or an end-user if they are allowed to.

Here is a quick glossary of terms so that we are all on the same page here:

* Client applications - application (mobile/web/background service) requesting access to data on behalf of the user
* Resource application - the application or API that exposes data or functionality
* Permissions - the ability for a client application to perform some action on data owned by a resource application (e.g. read emails in Exchange Online through Microsoft Graph API)
* Consent prompt - the pop-up where a user is asked to grant an application the requested permissions
* Consent grant - the result of saying "yes" to the consent prompt
* Admin Consent - a slightly different method of giving consent within an organisation. An admin can grant an application access to requested permissions that cannot be granted by a regular user. Usually these would be dangerous permissions. However, admin consent can also be done to automatically consent to given permissions to an application for all users in an organisation(as such end-users would not have to faff around with granting consent themselves).
* Applications - the definition of an application in Azure AD (Application registration)
* Service principal - the representation of an app in a given tenant (Azure AD Enterprise application)

There are two types of permissions in Azure AD, delegated and application permissions:

* Delegated permissions
  * More often used permission model.
  * Used by apps that have a signed-in user present in order to make calls on behalf of that user
  * Can be consented to by non-admin users, but high-privileged permissions will likely require explicit admin consent first.
  * "Effective permissions" are the intersection of the User's permissions and what the application has been granted consent to do. (e.g. you might have the ability to read, write to OneDrive, but the application would only ask for read access. As such effectively, it only has your read permission to your OneDrive)
* Application permissions
  * Slightly less often, but definitely more dangerous
  * used by apps that DO NOT require a signed-in user present (so imagine a daemon or background service)
  * application has permissions to do what it was consented to (e.g. if you give it Read to the OneDrive service, it has read access to OneDrive as a whole. So any user in the organisation)
  * Application permissions always require admin consent

Back to the applications now, it is key to mind the service principals in your tenant. Applications can be authorised to access data and as such, an application's service principal is also its security principal. Service principals can be granted access to various bits in the tenant using the following methods:

* Azure role assignment
* Azure AD role assignment
* Owner of group, application or service principal
* App-only permissions grants
* Delegated permission grants
* Azure Key Vault ACLs

Ultimately, applications can be an effective way of getting access to an organisation as an initial foothold. This is known as "consent phishing" where users are targeted with seemingly innocuous attachments that ask users to consent to application permissions. All in all the attack plays out as follows:

1. An attacker registered an app with an OAuth 2.0 provider such as Azure AD
2. App is configured in a way to make it look trustworthy, like using the name of a popular product.
3. Attacker gets a link in front of users, which can be done in various ways
4. User clicks the link and is shown an authentic consent prompt asking them to grant the app permission to data
5. If a user clicks accept, they grant the app permissions to access sensitive data
6. The app gets an authorization code which it redeems for an access token, and potentially a refresh token
7. The access token is used to make API calls on behalf of the user.

The key questions to ask yourself on an engagement and review of cloud apps in the following:

* What is this? - where did this app come from, and who assigned it to people
* Who is this assigned to? - if you know what this app is, then ask yourself whether it has been assigned to the correct people
* What are these permissions? - if you know about this app, then are you aware of what sort of permissions it's asking from users in the directory

So as a start, it would be useful to establish what high-privileged permissions are in use within an organisation's estate and make a list of them. As with a lot of stuff, these ones should be already expected, but it's always worth being safe and if you do find something that they aren't aware of it might be cause for concern. the following permissions are some stuff to look out for:

* Mail.*
* Mail.Send
* MailboxSettings.*
* Contacts.*
* People.*
* Files.*
* Notes.*
* Directory.AccessAsUser.All
* Directory.ReadWrite.All
* Application.ReadWrite.All
* Domain.ReadWrite.All
* EduRoster.ReadWrite.All
* Group.ReadWrite.All
* Member.Read.Hidden
* RoleManagement.ReadWrite.Directory
* User.ReadWrite.All
* User.ManageCreds.All
* user_impersonation

Some common low-level permission are the following which should constitute an almost minimal required set of perms in order to allow apps to use SSO for login:

* User.Read
* open_id
* email
* profile

So you're on an engagement and you want to have a look for suspicious apps. Well there are two general options. If there aren't that many apps and you know what you're looking for, then perhaps the most thorough way would be to go through the audit logs for apps and look for signs of suspicious activity. Stuff like "IsAdminConsent" set to True which indicated a global admin has granted an app broad permissions to data. Go through all the apps in the "Enterprise Apps" section in Azure AD and check audit logs.

If there are a decent amount of apps and you don't feel as comfy doing an assessment of each one individually, then [this script](https://aka.ms/getazureadpermissions) can be pretty useful. The script is made by Microsoft people and it generated a CSV file with all the apps and permissions assigned to them as well as marks potentially dangerous permissions for you.

Using it you can start off with all the marked high risk applications and compare against user assigned count for those apps. Be weary of apps marked as high risk and where users are set to AllUsers. Although this could be common for Microsoft apps, any other third-party apps should be carefully reviewed. Also review the "HighRiskUsers" tab where it lists all the users with high privilege or access to sensitive information. Try to check if there are any you probably don't want to have access to that (e.g. H.R, financials, etc.). Lastly, check the actual permissions assigned to each application. Look for any of the above listed dangerous permissions and verify if these are necessary.

To try to remediate these issues you have a couple of options but usually a combination of all of them would be the best way to harden this potential attack vector:

## Set Policies

Use application consent policies to limit user consent to applications. One common scenario is to allow users to only consent to Microsoft verified applications or publishers requesting low risk permissions. These permissions are defined by the administrator of the tenant and can be modified based on company requirements. In addition, if an organisation is already using Microsoft Cloud App Security, then they can set an app permissions policy that would automatically revoke an app or a specific user from an app when risk is detected.

## Risk-based user step-up consent (enabled by default)

When a risky request is detected, the request will be "stepped up" to require admin approval. Users will see the warning, but an admin will have to approve it. Admins should have a process in place when these requests come in, that they don’t just hang there – it is important to assign owners and take action based on what you see in your audit logs.  

## Detect risky OAuth applications

As a start, it would be useful for the organisation to frequently audit applications in the directory. One simple solution would be running the script from earlier, but that still involves some manual work. A better solution would be to configure some Azure Monitor alerts to send notifications to admins when an OAuth app has reached some criteria such as requiring high permissions or being authorized by more than 50 users. Ultimately the best solution would be to use Microsoft Cloud App Security and perform frequent hunts for dangerous apps.

## Developer training

Although not really a useful recommendation for your run of the mill engagements. This ultimately would be a really useful one for organisations that have a lot of internal apps with dangerous permissions enabled. Sometimes these apps would need these permissions but at the same time, it's possible that it's just devs not fully understanding the exact permissions they need and just asking for the easiest solution.

Here is a link for best practices for people using the Microsoft Graph API: <https://docs.microsoft.com/en-us/graph/best-practices-concept>
