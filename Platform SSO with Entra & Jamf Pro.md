# Platform SSO with Entra & Jamf Pro
## Overview
Platform Single Sign-On (Platform SSO or PSSO) is a method of joining a Mac computer to a domain that does not involve binding to the on-prem Active Directory server. Instead, PSSO registers the Mac with Entra ID to leverage IdP cloud services for user logins and profile synchronization. With PSSO, only verified domain accounts can log in, and the cached local accounts are created “on-demand” when a user signs into a mac for the first time.

Platform SSO is a framework built into computers with macOS 13 or later that provides users a simpler way to access organization resources protected by a cloud identity provider (IdP) (Microsoft Entra). It allows macOS to do the following:
- Pair and sync local account credentials with an identity provider password
- Create a secure, hardware bound non-phishable authentication factor to securely access organization resources
- Automatic sign-in to Microsoft apps like Outlook, Teams, OneDrive, etc.

<img width="631" height="453" alt="image" src="https://github.com/user-attachments/assets/aa99d5c1-4410-4fb1-9ec4-452fb209447a" />

## Jamf Configuration

### Single Sign-On Extension (Shared Device Keys + Password Method)

---

- **Payload Type:** `SSO`
- **Extension Identifier:** `com.microsoft.CompanyPortalMac.ssoextension`
  - ID for the Microsoft Enterprise Single Sign-On (SSO) plug-in used on managed macOS devices.
- **Team Identifier:** `UBF8T346G9`
  - Official Apple Developer Team ID assigned to Microsoft for its macOS software extensions and security plug-ins.
- **Sign-On Type:** `Redirect`
  - The SSO extension does not handle user passwords or credentials directly on the local machine. Instead, it uses a web-based or browser-driven authentication flow.

#### URLs

Identity providers where the app performs single sign-on.

The URLs must begin with `http://` or `https://` and must be unique across all configured Single Sign-On Extensions payloads. Query parameters and URL fragments are not allowed.

- `https://login.microsoftonline.com`
- `https://login.microsoft.com`
- `https://login.microsoftonline.us`
- `https://login-us.microsoftonline.com`
- `https://sts.windows.net`
- `https://login.partner.microsoftonline.cn`
- `https://login.chinacloudapi.cn`

---

### Platform SSO Setting
---
- **Use Platform SSO:** ✅ `Enabled`
- **Authentication Method:** `Password`
---
- **FileVault Policy (Apple silicon):** `Attempt`
  - `AttemptAuthentication` tries to synchronize and check the IdP if online but does not lock out the user due to a lack of connectivity if the local password is valid.
---
- **User Login Policy:** `Attempt`
---
- **Screensaver Unlock Policy:** Not configured
---
#### Non-Platform SSO Accounts
List of accounts that are not subject to the FileVault policy, user login policy, or screensaver unlock policy:
- **`[prestage]`**
  - PreStage managed local administrator account created during macOS Setup Assistant.
- **`[jamf binary]`**
  - Jamf-managed local administrator used for computers enrolled via PreStage enrollment or user-initiated enrollment.
  - This account is created by the Jamf management framework.
---
- **Registration Token:** Not configured
  - Reportedly only used for Intune MDM and has no effect in Jamf.
---
- **Quick Login for Temporary Session:** Not configured
  - Also known as **Authenticated Guest Mode (AGM)**.
  - Erases user data from select locations after each session, with full user home directory cleanup every eight hours.
  - Intended for shared environments with frequent short sessions.
  - We use a separate PSSO configuration profile for AGM Macs.
---
- **Enable Registration During Setup:** `Enable`
  - Enables creation of the first user account on the Mac during Setup Assistant.
  - Allows the Mac to register with Entra during out-of-box setup after Jamf enrollment.
- **New User Creation Authentication Method:** `Password`
---
- **Use Shared Device Keys:** `Enable`
  - Enables use of the same signing and encryption keys for all users.

> **Important:** This must be enabled for multi-user logins. All devices in our environment are considered "shared."
---
- **Create New User at Login:** `Enable`
  - Enables creation of new users at the login window using either a password or smart card.
---
- **Identity Provider Authorization:** `Enable`
  - Enables use of an identity provider during authorization prompts.
  - Extends SSO to system authorization prompts.
---
- **Account Display Name:** `[Company Entra]`
  - Display name for the account in notifications and authentication requests.
---
- **Login Frequency:** `60`
  - Frequency at which a complete login is required instead of a refresh.
  - The frequency must be one hour or more.
---
#### User Mapping
The attribute mapping used when creating new users or performing authorization.
- **Full Name:** `name`
- **Account Name:** `com.apple.PlatformSSO.AccountShortName`
> **Important:** `com.apple.PlatformSSO.AccountShortName` provides the necessary attribute mapping from the IdP so that user home directories are created without `@domain.com`.
> **Example:** `/Users/pparker` instead of `/Users/pparker@domain.com`
---
- **Account Authorization Type:** Not configured
  - Determines permissions for accounts during Platform SSO authentication after initial account creation.
  - `Standard` or leaving the setting unconfigured does not grant local administrator rights to users.
> **Potential future use:** This could allow domain administrator group members to approve local privilege escalation prompts, but Microsoft does not currently support this functionality.
---
- **New User Account Type:** Not configured
  - Determines permissions for accounts created at login by Platform SSO.
  - `Standard` or leaving the setting unconfigured does not grant local administrator rights to users.
> **Potential future use:** This could allow automatic assignment of administrator privileges to domain administrator group members, but Microsoft does not currently support this functionality.
---
- **Administrator Groups:** Not configured
- **Status:** Not yet supported
  - Defines the groups used for administrator access.
  - Membership is requested during authentication.
---
- **Additional Groups:** Not configured
- **Status:** Not yet supported
  - Defines groups that are created without administrator access.
---
- **Authorization Group:** Not configured
- **Status:** Not yet supported
  - Defines the pairing of authorization rights to group names.
  - The authorization right is updated to use the group when invoked.
---
- **Synchronize Profile Picture:** `Enable`
- **Status:** Not yet supported
  - Enables synchronization of the user's account profile picture from the IdP.
---
- **Device Identifiers in Attestation:** `Allow`
  - Allows the system to include the device UDID and serial number in Platform SSO attestations.
  - Ensures that hardware-bound proofs sent to the IdP (Entra ID) include core system attributes.
---
- **Authentication When Screen Is Locked:** `Do not handle`
  - Determines authentication behavior when a user is logged into the computer, the screen is locked, and an open application requires authentication.
  - Authentication requests continue without single sign-on.
  - This is the recommended configuration for ensuring native macOS lock screen and screensaver authorization rules operate normally.
---
## Custom Configuration

- **Custom Configuration:** `Custom PLIST`
  - PLIST containing key-value pairs for custom settings in this domain.
  - Extends SSO to specified websites and applications, including:
    - Microsoft
    - Apple
    - Jamf
    - Google
    - Adobe
    - Mozilla Firefox
    - Palo Alto Networks
    - Zoom

### Custom PLIST

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>AppPrefixAllowList</key>
    <string>com.microsoft.,com.apple.,com.jamf.,com.jamfsoftware.,com.google.,com.adobe.,org.mozilla.,com.paloaltonetworks.,us.zoom.config.</string>

    <key>browser_sso_interaction_enabled</key>
    <integer>1</integer>

    <key>disable_explicit_app_prompt</key>
    <integer>1</integer>

    <key>enable_se_key_biometric_policy</key>
    <false/>
</dict>
</plist>
```

---
## Associated Domains

This payload is needed within the Platform SSO Configuration Profile specifically when using Simplified Setup for Platform SSO in PreStage enrollments, but [do not enter any values in the App Identifier and Associated Domain fields](https://learn.jamf.com/r/en-US/jamf-school-documentation/Configuring_a_Platform_SSO_Profile_for_Computers_Microsoft_Entra_ID)
<img width="918" height="417" alt="image" src="https://github.com/user-attachments/assets/502fdc20-3d9b-44cb-8865-91ef0363092b" />

---


## Platform SSO PreStage with Simplified Setup

### 1. Upload the latest version of Company Portal to Jamf Pro > Settings > Packages
- Company Portal .pkg: https://go.microsoft.com/fwlink/?linkid=853070

---

### 2. Create a separate PreStage profile in Jamf Pro for Platform SSO
> **Important:** Devices assigned to this pre-stage will register with PSSO during the setup process (out-of-box experience or after factory reset)

