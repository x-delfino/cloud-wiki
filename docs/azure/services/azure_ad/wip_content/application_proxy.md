# Application Proxy

Application Proxy is in AAD feature that allows users to access on premises applications remotely leveraging AAD authentication. It's designed to replace the need for a VPN to access on prem apps. It supports the following types of apps:
- Web apps that use kerberos authentication
- Web apps that use forms or headers for authentication
- Web APIs
- Apps hosted behind a [Remote Desktop Gateway](https://learn.microsoft.com/en-us/azure/active-directory/app-proxy/application-proxy-integrate-with-remote-desktop-services)
- Apps integrated with Microsoft Authentication Library (MSAL)


## Kerberos

Application Proxy can use Kerberos Constrained Delegation (KCD) to support applications that rely on Kerberos authentication. The Application Proxy connectors are granted permission in AD to impersonate users, which allows for tokens to be sent and received on the users behalf. The authentication flow works roughly as follows:
- User attempts to access the application through the cloud Application Proxy.
- The Application Proxy redirects the request to AAD authentication services for preauthentication. Any policies such as Conditional Access are applied here. Once complete, AAD returns a token to the user.
- The user sends this token to the Application Proxy.
- The Application Proxy validates the token before the on-premise Connector then retrieves the User Principal Name (UPN) and Service Principal Name (SPN).
- The Connector performs KCD negotiation with on-premise AD, impersonating the user to get a Kerberos service ticket from AD.
- The Connector sends the original user request to the appication server along with the Kerberos service ticket.
- The application sends the response to the Connector which is sent back to the user via the Application Proxy.