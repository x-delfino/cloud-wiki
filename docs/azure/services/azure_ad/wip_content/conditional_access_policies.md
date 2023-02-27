# Conditional Access Policies

#### Conditional Access Policies (CAP)

Now if you've ever been greeted with a warning when attempting to login to an Azure AD account that says something like "oh you're not permitted to login with this device" or "oh you've signed in ok, but your device isn't allowed" then you've likely encountered a Conditional Access policy. At their core they are if-else statements that assess how a user attempts to access cloud resources and whether they can do so. Although simple as a concept, they can be surprisingly granular. Some general signals that CA policies can use to establish whether you can login is the following high-level areas:

* User or group membership
* IP address range
* Device platform
* Application
* Real-time and calculated risk detection - used in conjunction to analytics provided by Azure ID Identity Protection
* Microsoft Cloud App Security (MCAS) - enables user app access and sessions to be monitored and controlled in real time, increasing visibility and control over access to and activities performed within your cloud environment.

As for what decisions can come from this, you can either block on match or grant access on match. And in the event that you want to grant access you can still require some additional controls to be satisfied:

* Require MFA
* Require device to be marked as compliant
* Require Hybrid Azure AD joined device
* Require approved client app
* Require app protection policy

If you are in the process of analysing policies, then exercise an educated judgement on whether policies seem reasonable and any that do not, should be discussed with IT as for what business case exists for it. And if none, then raise it as an issue if you feel it adds a realistic level of risk of compromise.

^^^

Now if you've ever been greeted with a warning when attempting to login to an Azure AD account that says something like "oh you're not permitted to login with this device" or "oh you've signed in ok, but your device isn't allowed" then you've likely encountered a Conditional Access policy. At their core they are if-else statements that assess how a user attempts to access cloud resources and whether they can do so. Although simple as a concept, they can be surprisingly granular. Some general signals that CA policies can use to establish whether you can login is the following high-level areas:

* User or group membership
* IP address range
* Device platform
* Application
* Real-time and calculated risk detection - used in conjunction to analytics provided by Azure ID Identity Protection. Sign-in risks represent the chance that the specifc authentication request isn't legitimate, whereas User risks are typically based on credential leaks identified by Microsoft.
* Defender for Cloud Apps (previously Microsoft Cloud App Security) - enables user app access and sessions to be monitored and controlled in real time, increasing visibility and control over access to, and activities performed within, your cloud environment.

As for what decisions can come from this, you can either block on match or grant access on match. And in the event that you want to grant access you can still require some additional controls to be satisfied:

* Require MFA
* Require device to be marked as compliant
* Require Hybrid Azure AD joined device
* Require approved client app
* Require app protection policy
* Require acceptance of Terms of Use
* Require password change

If you are in the process of analysing policies, then exercise an educated judgement on whether policies seem reasonable and any that do not, should be discussed with IT as for what business case exists for it. And if none, then raise it as an issue if you feel it adds a realistic level of risk of compromise. There are a few best practices that Microsoft recommend for Conditional Access Policies:
- Set up emergency access accounts excluded from Conditional Access.
- Set up policies in report-only mode to monitor effect prior to enforcement.
- Block countries from which logins are never expected (exclude admins).

There are also some policy configurations that should be avoided as you could get locked out, in particular, policies that target All Users, All Cloud Apps and either:
- Block access
- Require device to be marked as compliant
- Require Hybrid AAD joined device
- Require app protection policy

## Continuous Access Evaluation
In the standard token flow, access tokens expire after an hour and so are refreshed with AAD. This allows for policies to be reevaluated to consider policy, user or resource changes before issuing a new token. However, relying on this process means there's a lag between state changes and policy enforcement. That's where Continously Access Evaluation (CAE) comes in. CAE implements a means for a compatible app or service to update the token issuer (AAD) about user state changes as soon as they occur. That means that the new policies can be enforced in near real time. 

## Security Defaults

Security Defaults are essentially a group of Conditional Access Policies that can be easily turned on, and actually come on by default on new AAD Tenants (since 10/2019). They are available to all licencing tiers including AAD free and were made available by Microsoft to ensure that a minimum security level could be achieved by all organisations that use AAD. Bear in mind, they cannot be used in conjunction with Conditional Access policies, and so don't often meet the requirements for larger organisations.

It's configuration is as simple as a `Yes` or `No` toggle will enable or disable the following controls:
- **Require all users to register for AAD MFA:** Users must register for AAD MFA within 14 days.
- **Require administrators to perform MFA:** Require MFA for all signins from users with the following roles:
	- Global Administrator
	- SharePoint Administrator
	- Exchange Administrator
	- Conditional Access Administrator
	- Security Administrator
	- Helpdesk Administrator
	- Billing Administrator
	- User Administrator
	- Authentication Administrator
- **Block legacy authentication protocols:** Block legacy authentication protocols as they do not support modern multifactor authentication.
- Require users to perform MFA 'when necessary'.
- Protect privileged actions. (eg. access to Azure portal)

## Identity Protection
> Requires AAD P2

AAD Identity Protection is a service to automate the detection and remediation of identity based risks. The service is powered by the information that Microsoft retrieves across the business, consumer and gaming spaces from which they analyse around 6.5 trillion signals per day. But what factors impact an identities risk level? A rough overview is as follows:

| Detection Type                | Description                                                                               |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| Anonymous IP address          | Sign in from an anonymised IP address such as TOR or a VPN service.                       |
| Atypical travel               | Sign in from a non-standard location based on user's recent activity.                     |
| Malware-linked IP address     | Sign in from an IP address associated with malware.                                       |
| Unfamiliar sign in properties | Sign in with non-standard properties based on user's recent activity.                     |
| Leaked credentials            | User's credentials have been leaked such as in a pastebin.                                |
| Password spray                | Multiple usernames are currently being attacked with specific passwords.                  |
| Azure AD threat intelligence  | Microsoft's internal and external threat intel sources have identitied an attack pattern. |

A number of additional detections are available Defender for Cloud Apps is in use:

| Detection Type                    | Description |
| --------------------------------- | ----------- |
| New Country                       |             |
| Activity from anoymous IP address |             |
| Suspicious inbox forwarding       |             |

Identity protection can also be used against service principals that will not be able to perform remedial actions such as MFA. These accounts can be protected using workload identity protection. The following types of risks can be flagged for these account types:

| Detection Type                                    | Description                                                                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Azure AD threat intelligence                      | Microsoft's internal and external threat intel sources have identitied an attack pattern.                                                                                                                                            |
| Suspicious signins                                | Signins with a non-standard property based on the baselines established for the service principal. Flagging changes in: <br/>&nbsp;IP Address<br/>&nbsp;ASN<br/>&nbsp;Target Resource<br/>&nbsp;User Agent<br/>&nbsp;Credential Type |
| Unusual additional of credentials to an OAuth app | Flagged when suspicious privileged credentials are added to an OAuth app which could indicate app compromise.                                                                                                                                                                                                                                     |
| Admin confirmed account compromised               | Account flagged as compromised by an administrative user using the Workload Identities UI or riskyServicePrincipals API.                                                                                                                                                                                                                                     |
| Leaked credentials                                | User's credentials have been leaked such as in a pastebin.                                                                                                                                                                           |

## Defender for Identity

Defender for Identity (formerly Azure Advanced Threat Protection (ATP))