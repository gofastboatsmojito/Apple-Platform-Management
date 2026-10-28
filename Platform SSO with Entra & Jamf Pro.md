# Platform SSO with Entra & Jamf Pro
## Overview
Platform Single Sign-On (Platform SSO or PSSO) is a method of joining a Mac computer to a domain that does not involve binding to the on-prem Active Directory server. Instead, PSSO registers the Mac with Entra ID to leverage IdP cloud services for user logins and profile synchronization. With PSSO, only verified domain accounts can log in, and the cached local accounts are created “on-demand” when a user signs into a mac for the first time.

Platform SSO is a framework built into computers with macOS 13 or later that provides users a simpler way to access organization resources protected by a cloud identity provider (IdP) (Microsoft Entra). It allows macOS to do the following:
- Pair and sync local account credentials with an identity provider password
- Create a secure, hardware bound non-phishable authentication factor to securely access organization resources
- Automatic sign-in to Microsoft apps like Outlook, Teams, OneDrive, etc.

...to be continued...
