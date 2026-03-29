---
name: pnp-ps-create-script
description: "Create PowerShell scripts using PnP PowerShell. Use when: writing automation scripts for Microsoft 365, building reusable PnP PowerShell scripts, scaffolding new scripts with parameters error handling and logging, following PowerShell best practices, creating production-ready M365 administration scripts."
argument-hint: "Describe what the script should do"
---

# Creating PnP PowerShell Scripts

Generate production-ready PowerShell scripts that use PnP PowerShell, following PowerShell community best practices.

## Prerequisites

The user must have PnP PowerShell installed and an Entra ID app registration configured. If not, use the `pnp-ps-setup` skill first. For cmdlet discovery, use the `pnp-ps-manage-m365` skill.

## Procedure

### Step 1: Gather Requirements

Ask the user:

1. **What should the script do?** — Specific M365 operations to perform
2. **Interactive or unattended?** — Will a user run it manually or will it run on a schedule/pipeline?
3. **Scope** — Single site, multiple sites, entire tenant?
4. **Input** — Where does the data come from? (parameters, CSV file, SharePoint list, hardcoded)
5. **Output** — What should the script produce? (console output, CSV report, log file, modifications to M365)

### Step 2: Scaffold the Script

Use the following structure as a template. All scripts MUST include these elements:

```powershell
#Requires -Modules PnP.PowerShell

<#
.SYNOPSIS
    Brief description of what the script does.

.DESCRIPTION
    Detailed description of the script's purpose, behavior, and any prerequisites.

.PARAMETER SiteUrl
    The URL of the SharePoint site to connect to.

.PARAMETER ClientId
    The Client ID of the Entra ID application registration.

.EXAMPLE
    .\Script-Name.ps1 -SiteUrl "https://contoso.sharepoint.com/sites/hr" -ClientId "00000000-0000-0000-0000-000000000000"

.NOTES
    Author:  <author>
    Version: 1.0
    Date:    <date>
#>

[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [ValidateNotNullOrEmpty()]
    [string]$SiteUrl,

    [Parameter(Mandatory)]
    [ValidateNotNullOrEmpty()]
    [string]$ClientId
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

# --- Functions ---

# --- Script Blocks ---

begin {
    # Initialize transcript logging
    $TranscriptPath = Join-Path -Path $PSScriptRoot -ChildPath "logs"
    if (-not (Test-Path -Path $TranscriptPath)) {
        New-Item -Path $TranscriptPath -ItemType Directory -Force | Out-Null
    }
    $TranscriptFile = Join-Path -Path $TranscriptPath -ChildPath "$(Get-Date -Format 'yyyyMMdd_HHmmss')_$($MyInvocation.MyCommand.Name -replace '\.ps1$','').log"
    Start-Transcript -Path $TranscriptFile -Append

    Write-Verbose "Script started at $(Get-Date -Format 'o')"
    Write-Verbose "Connecting to $SiteUrl"

    try {
        Connect-PnPOnline -Url $SiteUrl -Interactive -ClientId $ClientId
        Write-Verbose "Connected successfully"
    }
    catch {
        Write-Error "Failed to connect to $SiteUrl: $_"
        Stop-Transcript
        throw
    }
}

process {
    try {
        # Script logic here
    }
    catch {
        Write-Error "Script failed: $_"
        throw
    }
}

end {
    Disconnect-PnPOnline -ErrorAction SilentlyContinue
    Write-Verbose "Script completed at $(Get-Date -Format 'o')"
    Stop-Transcript
}
```

### Step 3: Apply PowerShell Best Practices

Every generated script MUST follow these rules:

#### Naming & Structure

- **Use approved verbs** for function names: `Get-`, `Set-`, `New-`, `Remove-`, `Import-`, `Export-`, etc. Run `Get-Verb` for the full list.
- **Use PascalCase** for function names, parameters, and variables.
- **Use `Verb-Noun` naming** with a project-specific prefix for custom functions (e.g., `Get-ProjectSiteData`).
- **One script = one purpose.** Break complex workflows into multiple scripts or functions.

#### Parameters

- Use `[CmdletBinding()]` on all scripts and advanced functions.
- Add `SupportsShouldProcess` when the script makes changes — this enables `-WhatIf` and `-Confirm`.
- Mark required parameters with `[Parameter(Mandatory)]`.
- Add `[ValidateNotNullOrEmpty()]`, `[ValidateSet()]`, `[ValidatePattern()]`, or `[ValidateRange()]` where appropriate.
- Use typed parameters (`[string]`, `[int]`, `[switch]`, `[string[]]`).
- Provide sensible defaults for optional parameters.

#### Error Handling

- Set `$ErrorActionPreference = 'Stop'` at the top to catch all errors.
- Use `begin/process/end` script blocks to separate initialization, logic, and cleanup.
- Wrap logic in `try/catch` within each block.
- Always disconnect and stop transcript in the `end` block.
- Use `Write-Error` for errors, never `Write-Host` for error output.
- For non-terminating errors in loops, use `-ErrorAction Stop` on individual cmdlets or handle with `-ErrorAction SilentlyContinue` and check `$?`.

#### Output & Logging

- Use `Start-Transcript` / `Stop-Transcript` in `begin` / `end` blocks to capture a full session log.
- Store transcript files in a `logs/` subfolder next to the script, with timestamped filenames.
- Use `Write-Verbose` for detailed progress information (visible with `-Verbose`).
- Use `Write-Information` for status messages.
- Use `Write-Warning` for potential issues.
- Use `Write-Progress` for long-running operations with many items.
- **Never use `Write-Host`** for data output — it bypasses the pipeline. Use `Write-Output` or return objects directly.
- Return structured objects, not formatted strings — let the caller decide formatting.

#### PnP PowerShell Specifics

- Always `Disconnect-PnPOnline` in the `end` block.
- Use `-Connection` parameter when working with multiple site connections.
- Use `New-PnPBatch` / `Invoke-PnPBatch` for bulk operations (50+ items).
- Add `-Verbose` support by using `Write-Verbose` throughout to aid debugging.
- When looping over many items, include `Write-Progress` with percentage.

### Step 4: Choose the Authentication Pattern

Based on the user's requirements from Step 1:

#### Interactive (user runs manually)

```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [string]$SiteUrl,

    [Parameter(Mandatory)]
    [string]$ClientId
)

Connect-PnPOnline -Url $SiteUrl -Interactive -ClientId $ClientId
```

#### App-Only with Certificate (scheduled/unattended)

```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [string]$SiteUrl,

    [Parameter(Mandatory)]
    [string]$ClientId,

    [Parameter(Mandatory)]
    [string]$Tenant,

    [Parameter(Mandatory)]
    [string]$CertificatePath
)

Connect-PnPOnline -Url $SiteUrl -ClientId $ClientId -Tenant $Tenant -CertificatePath $CertificatePath
```

#### App-Only with Base64 Certificate (Azure Functions / pipelines)

```powershell
Connect-PnPOnline -Url $SiteUrl -ClientId $ClientId -Tenant $Tenant -CertificateBase64Encoded $CertBase64
```

### Step 5: Implement the Script Logic

Follow these patterns when building the script body:

#### Processing Collections

```powershell
$items = Get-PnPListItem -List $ListName -PageSize 500
$total = $items.Count
$current = 0

foreach ($item in $items) {
    $current++
    Write-Progress -Activity "Processing items" -Status "$current of $total" -PercentComplete (($current / $total) * 100)

    if ($PSCmdlet.ShouldProcess($item["Title"], "Update item")) {
        Set-PnPListItem -List $ListName -Identity $item.Id -Values @{ "Status" = "Processed" }
    }
}

Write-Progress -Activity "Processing items" -Completed
```

#### Bulk Operations with Batching

```powershell
$batch = New-PnPBatch
$items = Import-Csv -Path $CsvPath

foreach ($item in $items) {
    Add-PnPListItem -List $ListName -Values @{
        "Title"  = $item.Title
        "Email"  = $item.Email
    } -Batch $batch
}

Invoke-PnPBatch -Batch $batch
Write-Verbose "Batch complete: $($items.Count) items processed"
```

#### Multi-Site Operations

```powershell
$sites = Get-PnPTenantSite -Filter "Url -like '/sites/project-'"

foreach ($site in $sites) {
    Write-Verbose "Processing site: $($site.Url)"

    try {
        Connect-PnPOnline -Url $site.Url -Interactive -ClientId $ClientId
        # Per-site logic here
    }
    catch {
        Write-Warning "Failed to process $($site.Url): $_"
        continue
    }
}

# Reconnect to original site if needed
Connect-PnPOnline -Url $SiteUrl -Interactive -ClientId $ClientId
```

#### Generating Reports

```powershell
$results = [System.Collections.Generic.List[PSCustomObject]]::new()

foreach ($site in $sites) {
    $results.Add([PSCustomObject]@{
        SiteUrl   = $site.Url
        Title     = $site.Title
        Storage   = $site.StorageUsageCurrent
        Owner     = $site.Owner
    })
}

if ($OutputPath) {
    $results | Export-Csv -Path $OutputPath -NoTypeInformation -Encoding UTF8
    Write-Verbose "Report exported to $OutputPath"
}
else {
    $results
}
```

### Step 6: Validate the Script

Before delivering the script, verify:

- [ ] Has comment-based help (`.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`)
- [ ] Uses `[CmdletBinding()]` and typed parameters with validation
- [ ] Sets `$ErrorActionPreference = 'Stop'` and `Set-StrictMode -Version Latest`
- [ ] Uses `begin/process/end` blocks with `Disconnect-PnPOnline` and `Stop-Transcript` in `end`
- [ ] Uses `SupportsShouldProcess` if it modifies data
- [ ] Uses `Write-Verbose` / `Write-Progress` instead of `Write-Host`
- [ ] Returns objects, not formatted strings
- [ ] Uses batching for bulk operations (50+ items)
- [ ] Has `#Requires` statements for PowerShell version and modules
- [ ] Contains no hardcoded URLs, credentials, or tenant-specific values