# netapp-ontap-powershell

PowerShell scripts for NetApp ONTAP storage automation and administration.

## Contents

| File | Purpose |
|------|---------|
| `Install_NetAPP_NFS_Plugin.ps1` | Installs the NetApp NFS Plug-in for VMware VAAI per ESXi host |
| `NetApp-Disk-Serials.txt` | Disk serial number reference |
| `ONTAP-Initial-Cluster-Setup.txt` | Initial cluster setup command reference |

## Prerequisites

- PowerShell 5.1+ or PowerShell 7+
- `DataONTAP` module: install from NetApp Support or PowerShell Gallery
- VMware PowerCLI (for `Install_NetAPP_NFS_Plugin.ps1`)
- ONTAP cluster with management LIF accessible

## Quick Start

```powershell
Import-Module DataONTAP
Connect-NcController -Name <cluster-mgmt-ip> -Credential (Get-Credential)
Get-NcNode
```

## CI/CD

All PRs validated by PSScriptAnalyzer, secret scan, and header compliance.

## Owner

humbledgeeks-allen | [HumbledGeeks.com](https://humbledgeeks.com)
