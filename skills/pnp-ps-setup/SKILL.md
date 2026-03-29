---
name: pnp-ps-setup
description: "Set up PnP PowerShell environment. Use when: installing PnP PowerShell module, registering Entra ID app for PnP PowerShell, configuring permissions, authenticating to Microsoft 365 tenant, connecting with Connect-PnPOnline, setting up app-only or delegated access."
argument-hint: "Describe what you need: install module, register app, configure permissions, or authenticate"
---

# PnP PowerShell Setup

Set up the complete PnP PowerShell environment: module installation, Entra ID app registration, permission configuration, and authentication.

## Procedure

Follow the stages below in order. Skip stages the user has already completed.

### Stage 1: Module Installation

1. Check if PnP.PowerShell is already installed:

```powershell
Get-Module PnP.PowerShell -ListAvailable
```

2. Check PowerShell version — PnP PowerShell requires **PowerShell 7.4.0 or later**:

```powershell
$PSVersionTable.PSVersion
```

If PowerShell version is below 7.4.0, inform the user they need to upgrade. Link: https://learn.microsoft.com/powershell/scripting/install/installing-powershell

3. If the module is not installed, install the stable build:

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser
```

If the user wants the nightly/prerelease build instead:

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser -AllowPrerelease -SkipPublisherCheck
```

4. If the module is already installed, offer to update it:

```powershell
Update-Module PnP.PowerShell -Scope CurrentUser
```

### Stage 2: Entra ID App Registration

> Every PnP PowerShell connection requires a custom Entra ID Application Registration. This is mandatory since September 9, 2024.

**Ask the user before proceeding:**

1. **Access type** — Will this be used for:
   - **Delegated (Interactive)** — The user will log in interactively each time (suitable for manual script execution)
   - **App Only** — Scripts will run unattended without user intervention (suitable for scheduled/automated tasks)

2. **tenant name** (e.g., `contoso.onmicrosoft.com`)

3. **Permission scopes** — What operations will be performed? Use this table to guide the user:

| Scenario | Access Type | Recommended Permissions |
|----------|-------------|------------------------|
| Read SharePoint content | Delegated | SharePoint > AllSites.Read |
| Write SharePoint content | Delegated | SharePoint > AllSites.Write |
| Manage SharePoint sites | Delegated | SharePoint > AllSites.Manage |
| Full SharePoint control | Delegated | SharePoint > AllSites.FullControl |
| Read specific sites (app) | App Only | SharePoint > Sites.Selected |
| Read all sites (app) | App Only | SharePoint > Sites.Read.All |
| Write all sites (app) | App Only | SharePoint > Sites.ReadWrite.All |
| Full SharePoint control (app) | App Only | SharePoint > Sites.FullControl.All |
| Microsoft Graph operations | Either | Use `-Verbose` on cmdlets or consult docs for specific permissions |
| Power Platform operations | Delegated | Azure Service Management > user_impersonation AND Dynamics CRM > user_impersonation AND PowerApps Service > User |

Ask the user which scopes they need. Recommend starting with minimum permissions and expanding as needed.

#### Option A: Delegated (Interactive) App Registration

Use the automatic registration cmdlet. The user needs at least the **Application Developer** role in Entra ID. **Global Administrator** may be needed to consent to permissions.

```powershell
Register-PnPEntraIDAppForInteractiveLogin -ApplicationName "PnP.PowerShell" -Tenant [yourtenant].onmicrosoft.com
```

To specify custom permission scopes, add the relevant parameters:
- `-GraphApplicationPermissions` 
- `-GraphDelegatePermissions`
- `-SharePointApplicationPermissions`
- `-SharePointDelegatePermissions`

Example with custom SharePoint delegated permissions:

```powershell
Register-PnPEntraIDAppForInteractiveLogin -ApplicationName "PnP.PowerShell" -Tenant [yourtenant].onmicrosoft.com -SharePointDelegatePermissions "AllSites.Write"
```

After registration, the user will be prompted to consent to the permissions. Take note of the **Application (client) ID** — it is needed for connecting.

#### Option B: App Only App Registration

This creates a certificate-based app registration for unattended script execution.

```powershell
$result = Register-PnPEntraIDApp -ApplicationName "PnP.PowerShell" -Tenant [yourtenant].onmicrosoft.com -OutPath c:\mycertificates -DeviceLogin
$result
```

This will:
- Register the app in Entra ID
- Generate a certificate (.cer and .pfx) at the specified `-OutPath`
- Upload the public key to the app registration
- Display a URL for consent — the user must navigate to it and grant consent

To specify custom permission scopes, add the same permission parameters as Option A.

If using `Sites.Selected` permission, the user must also grant per-site access:

```powershell
Grant-PnPAzureADAppSitePermission -AppId "<Client ID>" -DisplayName "PnP PowerShell" -Permissions Read -Site <SharePoint site URL>
```

> Running `Grant-PnPAzureADAppSitePermission` requires connecting with a **different** app registration that has `AllSites.FullControl` delegated permission, logged in as a Global or SharePoint Administrator.

### Stage 3: Authentication

Based on the app registration type created in Stage 2, guide the user to connect.

**Ask the user** for:
- Their SharePoint Online tenant URL (e.g., `contoso.sharepoint.com`)
- The **Client ID** from the app registration created in Stage 2

#### For Delegated (Interactive) Apps

**Interactive login** (recommended for most manual use):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -Interactive -ClientId <clientid>
```

**Device login** (when authenticating from another device or no GUI browser available):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -DeviceLogin -ClientId <clientid>
```

**Web Account Manager** (Windows 10 1703+ only — supports Windows Hello, FIDO keys, SSO):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -OSLogin -ClientId <clientid>
```

**Username/password** (no MFA support — less recommended):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -ClientId <clientid> -Credentials (Get-Credential)
```

#### For App Only Apps

The user needs the **Client ID**, **tenant name**, and the **certificate** from Stage 2.

**Using certificate file (.pfx)**:

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -ClientId <clientid> -Tenant [yourtenant].onmicrosoft.com -CertificatePath <path to .pfx>
```

If the .pfx has a password, add:
`-CertificatePassword (ConvertTo-SecureString -AsPlainText 'password' -Force)`

**Using certificate thumbprint** (certificate stored in Windows Certificate Store):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -ClientId <clientid> -Tenant [yourtenant].onmicrosoft.com -Thumbprint <certificate thumbprint>
```

**Using base64-encoded certificate** (common for Azure Functions):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -ClientId <clientid> -Tenant [yourtenant].onmicrosoft.com -CertificateBase64Encoded <base64 string>
```

#### Validate the Connection

After connecting, validate:

```powershell
Get-PnPConnection
```

### GCC / National Cloud Environments

If the user operates in a GCC or national cloud, add `-AzureEnvironment` to both registration and connection cmdlets:

```
-AzureEnvironment [USGovernment|USGovernmentHigh|USGovernmentDoD|Germany|China]
```

## Completion Criteria

- PnP.PowerShell module is installed and available
- Entra ID app registration is created with appropriate permissions
- User has successfully connected using `Connect-PnPOnline`
- Connection validated with `Get-PnPConnection`