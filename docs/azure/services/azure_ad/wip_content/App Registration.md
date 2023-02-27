## Application Registration

Applications in AAD are represented by either Application Objects or Service Prinicpals.

Application Objects are created using the App Registration functionality in AAD. These represent the application and define how tokens can be issued to it by AAD.

Service principals are identities 

An application has one application object associated with it and housed within it's home AAD. This application object can then be referenced by one of more service principals in each of the tenants that the application is being used, including the home directory.

Not all service principals reference back to an application object; Microsoft recommends that they are, however they can be used in a way more akin to traditional on-prem AD service accounts.

Comparing this to traditional AD, it's good to realise that application isn't really a thing on prem. There's nothing stopping an end user from setting up an application to be authentication ?? check this.

Registered applications can be available to a number of audiences:
- Single Tenant
- Multitenant
- Multitenant and personal Microsoft Accounts
- Personal Microsoft Accounts only

App registrations can be configured for a variety of target platforms:
- Web App
- Single Page App (SPA)
- iOS/macOS
- Android
- Mobile apps not using MSAL & Desktop Apps

Registered apps can be configured with credentials to allow them to authenticate with AAD either as a client certificate or secret; Microsoft recommend that secrets are not used in production.

API scopes can be defined to control access to data and functionality provided by the registered application. These can be configured as scopes that users or admins can consent to. These scopes will be included in OAUTH2 tokens when requested by applications/users with the necessary permissions.

Permission types are broken into two categories:
- **Delegated Permissions** - apps that perform actions as a present signed in user. The effecive permissions of the app will be the intersection of the delegated app permissions and the signed in user permissions. The app will never have more privileges than the signed in user.
- **Application Permissions** - for apps that run without a signed in user present. Only admins can consent to application permissions. This applicaiton will run with the full permissions granted as there is no signed in user to restrict the permissions to.
## OpenID Connect (OIDC) Scopes

AAD supprted the following OIDC scopes:
- `openid`: Required for sign in using OIDC by obtaining an identity token for authentication.
- `email`
- `profile`
- `offline_access`: Allows the application to retrieve refesh tokens to permit extended access to resources on behalf of the user.




`offline_access` and `user.read` are included in all application consents as they are typically required for application functionality.


Application roles can be used to assign application permissions to users. When users sign into the applications, the `roles` claim in the access token will include these role assignments and can therefore be used for authorization.



App governance can be enforced using Defender for Cloud Apps (used to be Microsoft Cloud App Security).