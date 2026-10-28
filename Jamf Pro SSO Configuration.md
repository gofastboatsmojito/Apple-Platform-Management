# Jamf SSO

This document describes the Single Sign-On (SSO) configuration used for Jamf, including the preferred OIDC-based SSO configuration and the legacy SAML-based configuration.

---

## Jamf Documentation

Reference documentation:

- [Identity and Access Integrations: Single Sign-On Options for Jamf Pro FAQ](https://learn.jamf.com/r/en-US/technical-articles/Single_Sign-On_Options_for_Jamf_Pro_FAQ)
- [SSO with OIDC Through Jamf Account](https://learn.jamf.com/r/en-US/jamf-pro-documentation-current/Jamf_SSO)
- [Configuring Microsoft Entra ID for SSO Integration with Jamf Account](https://learn.jamf.com/r/en-US/jamf-account-documentation/Microsoft_Entra_ID)

---

# OIDC-Based SSO

Jamf recommends using **OIDC-based SSO through Jamf Account**.

Whether users log in through Entra or with a Jamf ID, access is handled consistently, and support for additional platform features is enabled.

OIDC-based authentication through Jamf Account provides access to core features and modern platform services while maintaining appropriate tenant isolation.

Traditional authentication methods, such as local user accounts and **SAML-based SSO**, provide basic access to Jamf Pro but lack the secure context required for additional platform services, such as:

- Blueprints
- Compliance benchmarks

The following sections describe the minimum settings required for **OIDC-based SSO** to work in our environment.

---

## 1. Jamf Account Online (Jamf ID)

`account.jamf.com`

An administrator's online Jamf account must be registered with the organization, and the domain must be verified in our DNS records.

<img width="1728" height="873" alt="image" src="https://github.com/user-attachments/assets/5535b9b5-ebd2-4a71-b9da-0cc69b19cb37" />

### SSO Configuration

The following settings show how SSO should be configured.

> **Note:** The **Client ID** and **Client Secret** are obtained from the Microsoft Entra App Registration described in [Step 2](#2-microsoft-entra).

<img width="1259" height="1027" alt="image" src="https://github.com/user-attachments/assets/2d6aa972-c60c-492d-a741-cd76735478d5" />

<img width="994" height="984" alt="image" src="https://github.com/user-attachments/assets/9605ac45-b092-4f0b-a462-f00e727c5a44" />

<img width="992" height="961" alt="image" src="https://github.com/user-attachments/assets/b83b435f-1b48-4f7f-9ac1-81400a78359a" />


---

## 2. Microsoft Entra

Creating an **App Registration** in Microsoft Entra synchronizes the Identity Provider (IdP) with our organizational Jamf Account.

For Microsoft's Entra configuration, refer to:

[Configuring Microsoft Entra ID for SSO Integration with Jamf Account](https://learn.jamf.com/r/en-US/jamf-account-documentation/Microsoft_Entra_ID)

<!-- Screenshot: Jamf Account application/domain configuration -->
![Entra integration configuration](images/entra-integration.png)

### Enterprise App Registration

An Enterprise App Registration establishes the **OIDC-based connection**.

API permissions should be configured as shown below:

<!-- Screenshot: Entra API permissions -->
![Microsoft Entra API permissions](images/entra-api-permissions.png)

### Token Configuration

The following claims must be configured under **Token configuration** for the App Registration.

<!-- Screenshot: Entra token configuration -->
![Microsoft Entra token configuration](images/entra-token-configuration.png)

> **Important**
>
> The `upn` (`userPrincipalName`) claim is specific to our environment.
>
> This allows administrative accounts, such as `AD` or `EP` accounts, to log in to Jamf Pro even though these accounts do not have an email attribute because they are not mail-enabled.
>
> This setting directly corresponds to **Custom username claim name** in the Jamf Account configuration described in [Step 1](#1-jamf-account-online-jamf-id).

---

## 3. Jamf Pro

**Jamf Pro URL**

```text
company.jamfcloud.com
