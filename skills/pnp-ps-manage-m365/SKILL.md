---
name: pnp-ps-manage-m365
description: "Use PnP PowerShell cmdlets to manage Microsoft 365 services. Use when: managing SharePoint Online, Microsoft Teams channels users, Planner tasks plans, Entra ID groups users, Power Platform flows apps, Microsoft Graph, taxonomy, search, tenant settings, Viva Engage, Purview retention labels."
argument-hint: "Describe the M365 management task you want to perform"
user-invocable: true
---

# Using PnP PowerShell to Manage Microsoft 365

Guide for finding and using PnP PowerShell cmdlets to manage Microsoft 365.

## Prerequisites

The user must have PnP PowerShell installed and be connected to their tenant. If not, use the `pnp-ps-setup` skill first.

## Procedure

### Step 1: Identify the Service Area

Determine which Microsoft 365 service the user wants to manage. PnP PowerShell covers:

| Service Area | Cmdlet Prefix/Pattern | Examples |
|---|---|---|
| **SharePoint Online** | Site, List, File, Folder, Page, Web, ContentType, Field, View | `Get-PnPList`, `Add-PnPListItem`, `Set-PnPSite` |
| **Microsoft Teams** | Teams | `Get-PnPTeamsTeam`, `Add-PnPTeamsChannel`, `Add-PnPTeamsUser` |
| **Planner** | Planner | `Get-PnPPlannerPlan`, `Add-PnPPlannerTask`, `Get-PnPPlannerBucket` |
| **Entra ID (Azure AD)** | EntraID, AzureAD (alias) | `Get-PnPEntraIDUser`, `Get-PnPEntraIDGroup`, `New-PnPEntraIDGroup` |
| **Microsoft 365 Groups** | Microsoft365Group | `Get-PnPMicrosoft365Group`, `New-PnPMicrosoft365Group` |
| **Power Platform** | Flow, PowerApp, PowerPlatform | `Get-PnPFlow`, `Get-PnPPowerApp`, `Get-PnPPowerPlatformEnvironment` |
| **Microsoft Graph** | Graph, GraphMethod | `Invoke-PnPGraphMethod`, `Get-PnPGraphSubscription` |
| **Taxonomy / Term Store** | Term, Taxonomy | `Get-PnPTerm`, `New-PnPTermGroup`, `Get-PnPTermSet` |
| **Search** | Search | `Submit-PnPSearchQuery`, `Get-PnPSearchConfiguration` |
| **Tenant Administration** | Tenant | `Get-PnPTenantSite`, `Set-PnPTenant`, `New-PnPTenantSite` |
| **Viva / Engage** | Viva, VivaEngage | `Get-PnPVivaEngageCommunity`, `Set-PnPVivaConnectionsDashboardACE` |
| **Purview / Compliance** | RetentionLabel, SensitivityLabel | `Get-PnPRetentionLabel`, `Set-PnPFileRetentionLabel` |
| **Site Templates & Design** | SiteTemplate, SiteDesign, SiteScript | `Get-PnPSiteTemplate`, `Invoke-PnPSiteDesign` |
| **Hub Sites** | HubSite | `Get-PnPHubSite`, `Register-PnPHubSite` |
| **Apps / Add-ins** | App | `Get-PnPApp`, `Install-PnPApp`, `Add-PnPApp` |
| **User Profiles** | UserProfile | `Get-PnPUserProfileProperty`, `Set-PnPUserProfileProperty` |

### Step 2: Find the Right Cmdlet

PnP PowerShell follows the standard PowerShell verb-noun convention: `Verb-PnPNoun`.

**Common verbs and their purposes:**

| Verb | Purpose | Example |
|---|---|---|
| `Get` | Retrieve/read data | `Get-PnPList` |
| `Set` | Modify existing resource | `Set-PnPListItem` |
| `Add` | Add/create a resource or association | `Add-PnPListItem` |
| `New` | Create a new resource | `New-PnPList` |
| `Remove` | Delete a resource | `Remove-PnPListItem` |
| `Move` | Move a resource | `Move-PnPFile` |
| `Copy` | Copy a resource | `Copy-PnPFile` |
| `Invoke` | Execute an action | `Invoke-PnPSiteDesign` |
| `Export` | Export data | `Export-PnPTaxonomy` |
| `Import` | Import data | `Import-PnPTaxonomy` |
| `Enable`/`Disable` | Toggle feature state | `Enable-PnPFeature` |
| `Grant`/`Revoke` | Manage permissions | `Grant-PnPEntraIDAppSitePermission` |
| `Restore` | Recover deleted items | `Restore-PnPRecycleBinItem` |
| `Submit` | Submit for processing | `Submit-PnPSearchQuery` |
| `Register`/`Unregister` | Register or unregister | `Register-PnPHubSite` |

**To look up a cmdlet**, consult the official documentation:
- Cmdlet index: https://pnp.github.io/powershell/cmdlets/index.html
- Individual cmdlet: `https://pnp.github.io/powershell/cmdlets/<CmdletName>.html`

> **Note:** Some cmdlets marked with `2` in the docs are aliases for backwards compatibility (e.g., `AzureAD` cmdlets are aliases for the newer `EntraID` cmdlets). Prefer the non-alias version.

### Step 3: Get Cmdlet Details

For any cmdlet, look up its documentation page to find:
- **Synopsis** — what the cmdlet does
- **Syntax** — parameters and parameter sets
- **Examples** — usage examples
- **Required permissions** — what Entra ID app permissions are needed

The documentation URL pattern is:
```
https://pnp.github.io/powershell/cmdlets/<CmdletName>.html
```

or use the built-in help:

```powershell
Get-Help <CmdletName> -Full
Get-Help <CmdletName> -Examples
```

### Step 4: Execute the Command

Before executing, consider:

1. **Connection context** — The cmdlet runs against the site/tenant connected via `Connect-PnPOnline`. To target a different site, reconnect or use `-Connection` parameter if available.

2. **Permissions** — Ensure the app registration has the required permissions. If a cmdlet returns access denied, add `-Verbose` to see which permissions are needed:
   ```powershell
   Get-PnPListItem -List "Documents" -Verbose
   ```

3. **Batching** — For bulk operations, use `New-PnPBatch` and `Invoke-PnPBatch` to improve performance:
   ```powershell
   $batch = New-PnPBatch
   1..100 | ForEach-Object {
       Add-PnPListItem -List "MyList" -Values @{"Title" = "Item $_"} -Batch $batch
   }
   Invoke-PnPBatch -Batch $batch
   ```

4. **Destructive operations** — For `Remove-*`, `Clear-*`, or `Disable-*` cmdlets, ALWAYS ask the user for confirmation before executing.

### Step 5: Troubleshooting

| Problem | Solution |
|---|---|
| Access denied | Add `-Verbose` to see required permissions. Update app registration in Entra ID. |
| Cmdlet not found | Check module is imported: `Get-Module PnP.PowerShell`. Update module if needed. |
| Connection expired | Reconnect using `Connect-PnPOnline`. |
| Unexpected results | Check connection context with `Get-PnPConnection`. Ensure correct site URL. |
| Throttling | PnP PowerShell handles throttling automatically. For large batch operations, use `Invoke-PnPBatch`. |
