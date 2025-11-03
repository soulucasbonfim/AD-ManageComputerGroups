# AD-ManageComputerGroups
[![PowerShell](https://img.shields.io/badge/PowerShell-3.0%2B-blue.svg)](https://learn.microsoft.com/powershell/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20Server%20%7C%20Linux-lightgrey.svg)](#)
[![IaC](https://img.shields.io/badge/Infrastructure-as--Code-orange.svg)](#)
[![Version](https://img.shields.io/badge/version-1.9.7-brightgreen.svg)](#)

**Active Directory Group Reconciliation and Management Tool (Idempotent)**  
Version **1.9.7** — by [Lucas Bonfim de Oliveira Lima](https://linkedin.com/in/soulucasbonfim)

---

## Table of Contents
- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture & Flow](#-architecture--flow)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#-quick-start)
- [Configuration (INI as SSOT)](#%EF%B8%8F-configuration-ini-as-ssot)
- [Naming Conventions](#-naming-conventions)
- [Suggested OU Layout](#%EF%B8%8F-suggested-ou-layout)
- [Execution Flow & Modes](#-execution-flow--modes)
- [Logging Levels & Format](#-logging-levels--format)
- [Security & Permissions](#-security--permissions)
- [Troubleshooting](#-troubleshooting)
- [FAQ](#-faq)
- [Performance Tips](#%EF%B8%8F-performance-tips)
- [Limitations](#%EF%B8%8F-limitations)
- [Repository Structure](#%EF%B8%8F-repository-structure)
- [Versioning (Summary)](#-versioning-summary)
- [Contributing](#-contributing)
- [License](#%EF%B8%8F-license)
- [Author](#-author)

---

## 🚀 Overview
`AD-ManageComputerGroups` is a **fully INI-driven PowerShell automation framework** for **Active Directory (AD)**.  
It guarantees predictable, repeatable, and auditable lifecycle management of **per-host** and **infrastructure** security groups:

- **Per-host groups:** `GRP-ADM-<Host>`, `GRP-RDP-<Host>`, `GRP-SSH-<Host>`
- **Infrastructure groups:** `GRP-ADM-ALL-*`, `GRP-RDP-ALL-*`, `GRP-SSH-ALL-*`, `GRP-ADM-DENY-*`

All behavior is defined by a **single INI file** (Single Source of Truth, SSOT).  
No hardcoded logic. **Idempotent** by design. **Infrastructure-as-Code (IaC)** principles applied end-to-end.

---

## 🧠 Key Features
| Feature | Description |
|---|---|
| **Idempotent Runs** | Safe to re-execute — no duplication or side effects. |
| **100% INI-Driven** | WhatIf, OU creation, naming, flags, protection rules — all from INI. |
| **Cross-Platform Awareness** | Windows Servers, Linux Servers, Windows Workstations. |
| **OU Enforcement** | Ensures hierarchical OUs exist (optional auto-create). |
| **Reconciliation Engine** | Moves misplaced groups to the correct OU based on templates. |
| **Legacy & Orphan Cleanup** | Removes invalid, disabled, or orphaned groups safely. |
| **Simulation Mode** | `WhatIf=true` for dry-run audits without changing AD. |
| **Verbose Logging** | Timestamped, color-coded logs with levels and tags. |
| **Fail-Fast Validation** | Aborts immediately on missing mandatory keys. |

---

## 🧱 Architecture & Flow
```text
 ┌────────────────────────────────────────────────────┐
 │             AD-ManageComputerGroups.ps1            │
 │────────────────────────────────────────────────────│
 │ 1) Parse INI & validate (Fail-Fast)                │
 │ 2) Ensure OUs (hierarchical, optional)             │
 │ 3) Ensure ALL/DENY infra groups                    │
 │ 4) Enumerate computers (by OS type/state)          │
 │ 5) Create per-host ADM/RDP/SSH groups              │
 │ 6) Reconcile misplaced groups → correct OU         │
 │ 7) Cleanup orphan/disabled/mismatched groups       │
 │ 8) Log summary & reset WhatIf preference           │
 └────────────────────────────────────────────────────┘

```

## Requirements
- **PowerShell:** Windows PowerShell 3.0+  
- **Modules:** `ActiveDirectory` (RSAT AD DS Tools)  
- **Host:** Domain-joined Windows management host with network access to DCs  
- **Permissions:** Delegated rights to create/move/delete groups in the target OUs  
- **Active Directory:** Windows Server 2012+ (validated through 2022)  
- **Connectivity:** Stable LDAP/GC connectivity to domain controllers

---

## Installation
```powershell
# Install the Active Directory module (if missing)
Add-WindowsFeature RSAT-AD-PowerShell

# Clone repository
git clone https://github.com/soulucasbonfim/AD-ManageComputerGroups.git
cd AD-ManageComputerGroups

# (Optional) Unblock scripts if downloaded from the internet
Get-ChildItem -Recurse *.ps1 | Unblock-File

```

## ⚡ Quick Start

### 1. Clone the Repository  
Clone the repository locally and navigate to the directory:

**Command:**
git clone https://github.com/soulucasbonfim/AD-ManageComputerGroups.git
cd AD-ManageComputerGroups

---

### 2. Configure the INI File  
Open **AD-ManageComputerGroups.ini** and adjust it to match your Active Directory hierarchy:

**Example:**  
[Config.Core]  
WhatIf=false  
Verbose=true  
AutoCreateOUs=true  
AutoCreateGlobalGroups=true  
BaseReconciliationSearchDN=OU=Groups,OU=USER_CONTROL,OU=GLOBAL,DC=ODRL,DC=NET

**Tips:**  
- `WhatIf=true` → simulation mode (no modifications)  
- `AutoCreateOUs=true` → automatically creates missing Organizational Units  
- `AutoCreateGlobalGroups=true` → automatically creates ALL/DENY infra groups  
- `BaseReconciliationSearchDN` → defines the search root where reconciliation is performed

---

### 3. Validate the Active Directory Module  
Ensure the **ActiveDirectory** PowerShell module is installed and functional.

**Check:**  
Get-Module -ListAvailable ActiveDirectory  

**If not installed:**  
Add-WindowsFeature RSAT-AD-PowerShell

---

### 4. Run the Script  
Run directly with administrative privileges from a domain-joined host.

**Standard execution:**  
.\AD-ManageComputerGroups.ps1  

**Simulation mode (no changes):**  
.\AD-ManageComputerGroups.ps1 -WhatIf  

---

### 5. Interpret Log Output  
The script uses a structured, color-coded logging format for readability.

| Level | Description |
|-------|--------------|
| [INFO] | Informational message |
| [CREATE] | OU or group created |
| [SIMULATE] | Simulation (WhatIf mode) |
| [MOVE] | Group relocated to correct OU |
| [DELETE] | Invalid or orphan group removed |
| [CLEAN] | Legacy cleanup performed |
| [ERROR] | Critical error or exception |

**Example Output:**  
[15:02:11][START][GEN] Script started at 2025-11-03 15:02:11  
[15:02:14][CREATE][WIN-SRV] Creating: GRP-ADM-SRV01  
[15:02:15][MOVE][MOVE] Moving GRP-RDP-SRV02 from OU=RDP,OU=OLD to OU=RDP,OU=Windows,OU=Server  
[15:02:18][END][GEN] Script completed (Elapsed: 00:00:07.134)

---

### 6. Validate the Results  
After execution, verify that the expected groups and OUs exist.

**Expected Behavior:**  
- Per-host groups (GRP-ADM-*, GRP-RDP-*, GRP-SSH-*) are created automatically.  
- Global ALL/DENY groups are created or validated according to configuration.  
- Missing OUs are created recursively if enabled.  
- Disabled or orphaned groups are safely removed.

**Verification Commands:**  
Get-ADGroup -Filter 'Name -like "GRP-*"' | Select Name, DistinguishedName  

Get-ADComputer -Filter * -SearchBase "OU=Servers,OU=GLOBAL,DC=ODRL,DC=NET" |  
ForEach-Object {  
    "$($_.Name) → $(Get-ADGroup -Filter 'Name -like \"*' + $_.Name + '*\"').Name"  
}

---

## ⚙️ Configuration (INI as SSOT)

All configuration logic is centralized in **AD-ManageComputerGroups.ini**, serving as the **Single Source of Truth (SSOT)**.

**Characteristics:**  
- 100% of operational logic comes from INI definitions.  
- The script never hardcodes OUs, names, or descriptions.  
- Missing or invalid keys trigger immediate termination (fail-fast principle).  
- Supports global flags, OU mapping, naming templates, and exclusions.

**Key Sections:**  
- `[Config.Core]` → Core pipeline settings (WhatIf, Verbose, AutoCreate).  
- `[Pipeline.Flags]` → Controls which object types are processed.  
- `[Naming.Templates]` → Defines all naming conventions.  
- `[Descriptions.*]` → Provides group purpose text.  
- `[Infra.GlobalGroups.*]` → Defines ALL/DENY group configuration.  

---

## 🧩 Naming Conventions

**Per-host groups:**  
- Windows Server ADM: `GRP-ADM-{HOST}`  
- Windows Server RDP: `GRP-RDP-{HOST}`  
- Linux Server SSH: `GRP-SSH-{HOST}`  
- Workstation ADM: `GRP-ADM-{HOST}`  

**Infra groups:**  
- Global (ALL) scope: `GRP-ADM-ALL-WINDOWS-SERVERS`  
- Deny scope: `GRP-ADM-DENY-RDP`  

All names are standardized and automatically protected from cleanup.

---

## 🗂️ Suggested OU Layout

Recommended OU hierarchy to ensure organizational consistency:
- OU=Groups 
- └── OU=USER_CONTROL 
- └── OU=GLOBAL 
- ├── OU=Server 
- │ ├── OU=Windows 
- │ │ ├── OU=ADM 
- │ │ └── OU=RDP 
- │ └── OU=Linux 
- │ ├── OU=ADM 
- │ └── OU=SSH 
- └── OU=Workstation 
- ├── OU=ADM 
- └── OU=RDP



---

## 🔁 Execution Flow & Modes

**Execution Pipeline:**  
1. Parse and validate INI configuration.  
2. Ensure required OUs exist (hierarchical creation optional).  
3. Ensure ALL/DENY infra groups.  
4. Enumerate computer accounts by OS and role.  
5. Create per-host groups dynamically.  
6. Reconcile misplaced groups.  
7. Perform cleanup of orphan, disabled, or mismatched groups.  

**Modes:**  
- **Normal Mode:** Executes all changes.  
- **WhatIf Mode:** Simulates all actions without altering AD.  
- **Verbose Mode:** Adds extra detail for diagnostics.

---

## 🪵 Logging Levels & Format

Logs are structured, timestamped, and color-coded for clarity.

**Format:**  
`[HH:mm:ss][LEVEL][TAG] Message`

**Levels:**  
INFO, CREATE, SIMULATE, MOVE, DELETE, CLEAN, ERROR  

**Tags:**  
WIN-SRV, LIN-SRV, WIN-WS, MOVE, CLEAN, GEN  

Logs are console-based and fully compatible with transcript-based pipelines.

---

## 🔒 Security & Permissions

**Requirements:**  
- Domain-joined administrative host.  
- Delegated rights to manage groups and OUs.  
- RSAT AD PowerShell tools installed.  
- Network access to all domain controllers.

**Best Practices:**  
- Execute under a service account with constrained delegation.  
- Restrict write permissions to target OUs only.  
- Use `WhatIf=true` in new environments before production execution.

---

## 🧰 Troubleshooting

**Common Issues:**
| Symptom | Cause | Resolution |
|----------|--------|------------|
| Missing ActiveDirectory module | RSAT not installed | Run `Add-WindowsFeature RSAT-AD-PowerShell` |
| OU not found | AutoCreateOUs disabled | Enable AutoCreateOUs or manually create |
| No groups created | WhatIf=true | Set WhatIf=false |
| Groups not moving | Incorrect BaseReconciliationSearchDN | Adjust the DN path in INI |
| Cleanup skipped | Infra groups protected | Validate [Naming.Templates] section |

Logs provide immediate visual identification of skipped or failed actions.

---

## ❓ FAQ

**Q1: Can this script be re-executed safely?**  
Yes. It is fully idempotent and can run repeatedly without duplication.

**Q2: What happens if an OU is missing?**  
If `AutoCreateOUs=true`, it will be created automatically. Otherwise, the process stops with a warning.

**Q3: Are infra groups deleted during cleanup?**  
Never. Infra groups defined in the INI are protected automatically.

**Q4: Can I run this from a non-domain system?**  
No. The script requires a domain-joined host with AD connectivity.

---

## ⚙️ Performance Tips

- Limit LDAP search base with `ServerRootsDN` and `WorkstationRootsDN`.  
- Use specific name filters to speed up enumeration.  
- Avoid large-scale reconciliation during peak AD replication hours.  
- Run cleanup in off-peak windows for better throughput.

---

## ⚠️ Limitations

- Requires Windows PowerShell 3.0+ (not compatible with PowerShell Core).  
- Must run within a trusted AD environment.  
- Cannot manage cross-domain OUs.  
- Designed for on-prem AD only (no EntraID/AzureAD support).  

---

## 🗃️ Repository Structure
- AD-ManageComputerGroups/
- ├── AD-ManageComputerGroups.ps1
- ├── AD-ManageComputerGroups.ini
- ├── LICENSE
- ├── README.md
- └── Docs/
- ├── examples/
- ├── templates/
- └── changelog/


---

## 🧾 Versioning (Summary)

**Current Version:** 1.9.7  
**Release Date:** 2025-11-02  
**Change Highlights:**  
- Improved dynamic regex matching  
- Enhanced OU auto-discovery logic  
- Extended cleanup validation  
- Idempotency guarantees across reruns  

Versioning follows **Semantic Versioning (MAJOR.MINOR.PATCH)**.

---

## 🤝 Contributing

Contributions are welcome via pull requests.  
Please follow these rules:

1. Use feature branches.  
2. Keep commits atomic and descriptive.  
3. Update both script and INI examples.  
4. Validate with `WhatIf=true` before submitting.  
5. Follow PowerShell naming standards (PascalCase functions).

---

## ⚖️ License

Licensed under the **MIT License**.  
You are free to use, modify, and distribute this project with attribution.

---

## 👤 Author

**Developed by:**  
Lucas Bonfim de Oliveira Lima  
**LinkedIn:** [https://linkedin.com/in/soulucasbonfim](https://linkedin.com/in/soulucasbonfim)  
**GitHub:** [https://github.com/soulucasbonfim](https://github.com/soulucasbonfim)  
**Version:** 1.9.7 (2025-11-02)
