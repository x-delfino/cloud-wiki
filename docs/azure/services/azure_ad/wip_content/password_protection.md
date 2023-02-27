# Password Protection
> Requires AAD P1 Licensing (cloud-only users benefit from global banned password list on AAD Free)

Azure AD password protection is a handy service for avoiding weak passwords by enforcing global and custom banned password lists. Bear in mind, this is enforced only when passwords are set or changed, it does not audit existing passwords. The architecture of this solution in a hybrid environment is fairly straight forward, you have three main components:
- **Cloud service:** Service is managed from AAD.
- **On-premise proxy service:** Installed on a member server. It proxies communications between the DC agents and cloud service.
- **On-premise DC agent:** Installed on a Domain Controller. It stores, manages and enforces the banned password policy. Retrieving policy updates is initiated by this agent by requesting the updates from the proxy service.


## Smart Lockouts

Smart lockouts are an always on feature for all AAD tenants, which can be customised *if* AAD P1 is in use. By default smart lockouts do the following:
- Lock account for 1 minute after 10 failed signin attempts.
- Lock account for subsequent failed signins (following account lock as per above) with subsequent locks being for a longer period.
- Does not increment the bad-password counter on repeats of the same password attempt.
- *Familiar* and *unfamiliar* locations have independent attempt counters.

In hybrid environments leveraging Pass Through Authentication, smart lockouts can be used to avoid AD DS accounts from being locked by attackers. If licensed appropriately (AAD P1), the following settings can be configured for smart lockout:
- Lockout threshold
- Lockout duration




