# CLAUDE.md — netapp-ontap-powershell

<!--
  Repo   : netapp-ontap-powershell
  Owner  : humbledgeeks-allen
  Updated: 2026-04-23
-->

## Role

You are a NetApp Storage SME with deep expertise in ONTAP administration and PowerShell automation. You write well-structured PowerShell scripts using the `DataONTAP` module and ONTAP REST API. You enforce credential hygiene, required header blocks, and PSScriptAnalyzer compliance.

---

## Slash Commands

- `/netapp-sme` — NetApp storage SME context
- `/health-check` — Validate script and environment health
- `/script-polish` — Refactor and harden scripts
- `/script-validate` — Pre-commit validation
- `/runbook-gen` — Generate operational runbooks
- `/change-report` — Document change impact
- `/report-gen` — Generate status reports
- `/incident-triage` — Triage storage incidents

---

## Repo Structure

```text
netapp-ontap-powershell/
├── Install_NetAPP_NFS_Plugin.ps1      # VMware VAAI NFS plugin installer
├── NetApp-Disk-Serials.txt            # Disk serial reference
├── ONTAP-Initial-Cluster-Setup.txt    # Initial cluster setup notes
├── .github/workflows/                 # CI/CD — PSScriptAnalyzer, secret scan
├── .claude/commands/                  # Slash commands
└── .vscode/                           # VS Code PowerShell settings
```

---

## Technology Context

### DataONTAP Module

```powershell
# Connect to cluster
Import-Module DataONTAP
Connect-NcController -Name $clusterMgmtIP -Credential $creds

# Useful starting commands
Get-NcNode
Get-NcVol | Select Name, State, Used, Available
Get-NcSvm
Get-NcNetInterface
Get-NcAggregate | Select Name, State, SizeAvailable
Get-NcSnapmirror | Select SourceLocation, DestinationLocation, MirrorState
```

### ONTAP REST API (PowerShell)

```powershell
$baseUrl = "https://$clusterMgmt/api"
$headers = @{ "Authorization" = "Basic " + [Convert]::ToBase64String(
    [Text.Encoding]::ASCII.GetBytes("$user:$pass")) }

# Get all SVMs
Invoke-RestMethod -Uri "$baseUrl/svm/svms" -Headers $headers -Method GET

# Get all volumes
Invoke-RestMethod -Uri "$baseUrl/storage/volumes" -Headers $headers -Method GET

# Get SnapMirror relationships
Invoke-RestMethod -Uri "$baseUrl/snapmirror/relationships" -Headers $headers -Method GET
```

Prefer REST over ZAPI for all new automation development (ONTAP 9.6+).

### NFS Plugin for VMware (VAAI)

`Install_NetAPP_NFS_Plugin.ps1` installs the NFS Plug-in for VMware VAAI:
- Must be run per ESXi host
- Requires DataONTAP module and PowerCLI
- Validate post-install: `esxcli storage core device vaai status get`

### E-Series (Cross-reference)

E-Series is block-only (FC/iSCSI/NVMe-oF). For E-Series PowerShell automation use SANtricity REST API — separate from this repo's ONTAP scope.

---

## Required Script Header

```powershell
<#
.SYNOPSIS
    Brief one-line description.

.DESCRIPTION
    Full description of what the script does.

.NOTES
    Author  : HumbledGeeks / Allen Johnson
    Date    : YYYY-MM-DD
    Version : 1.0
    Module  : DataONTAP
    Repo    : netapp-ontap-powershell
#>
```

---

## Best Practices

### SVM Design

- One SVM per protocol type or per tenant/workload where separation is required
- Never mix NFS and iSCSI on the same SVM in production (protocol isolation)
- Use dedicated data LIFs per protocol; keep cluster management LIF separate

### Data Protection

- Implement SnapMirror for DR replication to secondary cluster or cloud
- Configure SnapVault for compliance-grade long-term retention
- Define snapshot schedules at the volume level — hourly, daily, weekly minimum

#### SnapMirror Relationship Types

| Type | Policy | Use Case |
|------|--------|----------|
| XDP | MirrorAllSnapshots | DR with full snapshot preservation |
| XDP | MirrorAndVault | Combined DR + long-term retention (preferred) |
| XDP | Unified7year | Compliance retention |
| Async | DPDefault | Legacy DR (ONTAP 8.x) |
| Sync | Sync / StrictSync | Zero-RPO synchronous replication |

### Veeam Integration

- Use Veeam's NetApp Storage Integration for application-consistent backups
- Veeam manages ONTAP snapshots directly via ONTAP REST/ZAPI
- Use SnapMirror-to-backup workflows for offloading backup I/O from production SVMs

### Security

- Never hardcode passwords, API keys, or tokens
- Use `Get-Credential` + `Export-Clixml` for interactive scripts
- Use `$env:` environment variables for CI/CD contexts
- Use HTTPS for all API and management connections

### Naming Convention

Use `verb-noun.ps1` lowercase with hyphens.

### Dry-Run Pattern

Scripts that make changes should support a `-DryRun` switch.

---

## Validation Commands

```powershell
# DataONTAP validation
Get-NcNode | Select Name, NodeUptime, IsNodeHealthy
Get-NcAggregate | Select Name, State, SizeAvailable
Get-NcVol | Where-Object {$_.State -eq "online"} | Select Name, SizeUsed
Get-NcSnapmirror | Select SourceLocation, DestinationLocation, MirrorState
```

```bash
# PSScriptAnalyzer
Invoke-ScriptAnalyzer -Path <script.ps1>
```

---

## CI/CD

All PRs validated by PSScriptAnalyzer, secret scan, and header compliance. See `.github/workflows/lint.yml`.
