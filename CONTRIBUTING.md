# Contributing to netapp-ontap-powershell

## PowerShell Standards

### Required Header Block

Every `.ps1` file must begin with a comment-based help block:

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

### Credential Rules

- Never hardcode passwords, API keys, or tokens
- Use `Get-Credential` + `Export-Clixml` for interactive scripts
- Use `$env:` environment variables for CI/CD contexts

### Design Rules

- Use HTTPS for all ONTAP management connections
- Prefer ONTAP REST API over ZAPI for ONTAP 9.6+
- Validate cluster connectivity before making changes

### Naming Convention

Use `verb-noun.ps1` lowercase with hyphens:
- `get-ontap-volumes.ps1`
- `set-snapmirror-policy.ps1`

### Dry-Run Pattern

Scripts that make changes should support a `-DryRun` switch.

## Commit Prefixes

`add`, `fix`, `docs`, `update`, `remove`, `refactor`

## Validation

Run `/script-validate <path>` before committing. The CI/CD pipeline runs PSScriptAnalyzer, secret scan, and header compliance on all PRs.
