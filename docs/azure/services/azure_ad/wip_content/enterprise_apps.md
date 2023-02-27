## Token Customisation

MS aim to optimise the size of tokens and so several claims included within ID tokens and access tokens are no longer present by default in v2.0 tokens. They must be added explicity on a per-application basis.

### Consent

By default, all users can consent to applications that don't require admin consent. e.g. a standard user can consent to applciations requesting access to their mailbox. This can help facilitate staff productivity but can introduce risk if not managed appropriately. There's a number of options that can be used to configure censent:

- Do not allow user consent
- Allow user consent for apps from verified publishers, for selected permissions
- Users can consent to all Apps
- Custom policy

Admin consent requests can be configured to allow end users to request admin consent to apps that they are unable to consent to. By default, Microsoft gflag 'risky' consent requests and elevates them to requiring admin approval. However, this control shouldn't be relied on.

### Single Sign On (SSO)

AAD can serve as the Identity Provider (IdP) for a wide range of apps. There are a large number available within the AAD App Gallery which require minimal configuration. Further than that, AAD supports SAML and OIDC based SSO. One of the great benefits of enabled SSO, is being able to centrally manage access to apps through tools such as Conditional Access Policies.

### User Provisioning

Users can be synchronised from AAD to third party applications. This minimizes administrative overhead when managing these applications but also comes with the following benefits:
- Users that are disabled in AAD can be automatically disabled in the third party application.
- Groups can be syncronised
- Centrally manage access to applications

System for Cross-domain Identity Management (SCIM) is one of the popular technologies that is often leveraged to achieve this. SCIM essentially defines two REST API endpoints: `/Users` and `/Groups` to provide a standardised mechanism for provisioning and deprovisioning identities across platforms. Infact, AAD Connect Cloud Sync actually uses SCIM to synchronise on-prem AD identities to AAD.