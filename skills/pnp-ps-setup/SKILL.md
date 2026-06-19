---
name: pnp-ps-setup
description: "Set up local machine environment for PnP PowerShell usage. Use when user wants to validate or setup local machine for PnP PowerShell usage. Stages included: installing PnP PowerShell module, registering Entra ID app for PnP PowerShell, configuring permissions, authenticating to Microsoft 365 tenant, connecting with Connect-PnPOnline, setting up app-only or delegated access."
---

# PnP PowerShell Setup

Set up local machine environment for PnP PowerShell usage. Stages included: module installation, Entra ID app registration, permission configuration, and authentication.

## Rules

- NEVER assume user intent, if instructions state to clarify user intent, ALWAYS ask the user for confirmation before proceeding.
- Answers MUST be short and concise, and MUST NOT include any extra information, just state what was done and what is the result.
- ALWAYS follow the procedure in the order specified!
- DO NOT fetch for additional information from web, instructions that are present here are enough. You MUST use them

## Procedure

Follow the stages below in order.

### Stage 1: Clarify user intent

Clarify if user wants to set up PnP PowerShell for the first time, or if they want to validate an existing setup. If the user wants to validate an existing setup, use the below steps as baseline checks that should be validated and confirmed and if any check would fail first, ALWAYS inform the user and wait for user confirmation before fixing it.

### Stage 2: Module Installation

1. Check if PnP.PowerShell is already installed:

```powershell
Get-Module PnP.PowerShell -ListAvailable
```

2. Check PowerShell version — PnP PowerShell requires **PowerShell 7.4.0 or later**:

```powershell
$PSVersionTable.PSVersion
```

If PowerShell version is below 7.4.0, inform the user they must upgrade before continuing and STOP. Do not proceed to module installation until the user confirms they have upgraded to PowerShell 7.4.0 or later. Windows PowerShell will not work. Link: https://learn.microsoft.com/powershell/scripting/install/installing-powershell

3. If the module is not installed, install the stable build:

If the user receives an **Untrusted repository** warning or error, instruct them to first run:

```powershell
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
```

Then install the module:

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

### Stage 3: Entra ID App Registration

> Every PnP PowerShell connection requires a custom Entra ID Application Registration. This is mandatory since September 9, 2024.

1. Validate if user environment variable `ENTRAID_CLIENT_ID` is set with app registration ID. If yes STOP and ALWAYS clarify with the user if this app should be used for the sign in. If not, proceed to create a new app registration.
If user confirms proceed to stage 4, otherwise proceed to create a new app registration.

**ALWAYS ASK THE USER BEFORE PROCEEDING:**

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

Ask the user which scopes they need. 

**THIS IS A MUST! ALWAYS CONFIRM WITH THE USER BEFORE PROCEEDING the access type, tenant name, and permission scopes!**

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

### Stage 4: Authentication

Based on the app registration type created in Stage 3, perform the authentication and validate it.

#### For Delegated (Interactive) Apps

**Interactive login** (recommended for most manual use):

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -Interactive -ClientId <clientid>
```

#### For App Only Apps

The user needs the **Client ID**, **tenant name**, and the **certificate** from Stage 2.

**Using certificate file (.pfx)**:

```powershell
Connect-PnPOnline [yourtenant].sharepoint.com -ClientId <clientid> -Tenant [yourtenant].onmicrosoft.com -CertificatePath <path to .pfx>
```

If the .pfx has a password, add:
`-CertificatePassword (ConvertTo-SecureString -AsPlainText 'password' -Force)`

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