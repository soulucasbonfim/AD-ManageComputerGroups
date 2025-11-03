# AD-ManageComputerGroups
**Active Directory Group Reconciliation and Management Tool (Idempotent)**  
Version: 1.9.7 — by [Lucas Bonfim de Oliveira Lima](https://linkedin.com/in/soulucasbonfim)

---

## 🚀 Overview
`AD-ManageComputerGroups` is an **enterprise-grade PowerShell automation framework** for managing per-host and infrastructure security groups in **Active Directory**.

The script dynamically **creates, reconciles, moves, and cleans** host-level (ADM/RDP/SSH) groups and infrastructure-wide (ALL/DENY) groups — all behavior driven entirely by a **single INI configuration file**, enabling **Infrastructure-as-Code (IaC)** for AD group management.

---

## ⚙️ Key Features
- **Idempotent Execution** – Safe to run multiple times without side effects.
- **Full INI-driven Logic** – No hardcoded values; 100% configuration-based.
- **Cross-Platform Awareness** – Windows Server, Linux Server, and Workstation support.
- **Automatic OU Discovery & Creation** – Optional hierarchical OU creation.
- **Legacy Cleanup** – Detects and removes orphaned, disabled, or misaligned groups.
- **Dynamic Reconciliation** – Moves misplaced groups to their correct OU.
- **Simulation Mode** – `WhatIf=true` allows safe dry-run auditing.

---

## 🧩 Configuration Model
All behavior is defined in a single file:  
`AD-ManageComputerGroups.ini` — serving as the **Single Source of Truth (SSOT)**.

### Example:
```ini
[Config.Core]
WhatIf=true
Verbose=true
AutoCreateOUs=true
AutoCreateGlobalGroups=true
BaseReconciliationSearchDN=OU=Groups,DC=example,DC=com
