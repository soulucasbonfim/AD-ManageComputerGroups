# AD-ManageComputerGroups
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-blue.svg)](https://learn.microsoft.com/powershell/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20Server%20%7C%20Linux-lightgrey.svg)](#)
[![Automation](https://img.shields.io/badge/type-Infrastructure--as--Code-orange.svg)](#)

**Active Directory Group Reconciliation and Management Tool (Idempotent)**  
Version **1.9.7** — by [Lucas Bonfim de Oliveira Lima](https://linkedin.com/in/soulucasbonfim)

---

## 🚀 Overview

`AD-ManageComputerGroups` is a **fully INI-driven PowerShell automation framework** that performs **lifecycle management** for host-level and infrastructure-level security groups in **Active Directory (AD)**.

It dynamically creates, reconciles, moves, and cleans up:
- **Per-host groups:** `GRP-ADM-<Host>`, `GRP-RDP-<Host>`, `GRP-SSH-<Host>`
- **Infrastructure groups:** `GRP-ADM-ALL-*`, `GRP-RDP-ALL-*`, `GRP-SSH-ALL-*`, and `GRP-ADM-DENY-*`

All logic is 100% defined by an **INI configuration file**, making the script **idempotent**, **auditable**, and **repeatable** — following strict *Infrastructure-as-Code (IaC)* principles.

---

## 🧠 Key Features

| Feature | Description |
|----------|-------------|
| **Idempotent Execution** | Safe to re-run multiple times with no duplication or side effects. |
| **INI-Driven Logic** | No hardcoded values — everything is configurable. |
| **OU Enforcement** | Ensures Organizational Units exist and are correctly structured. |
| **Cross-Platform Awareness** | Detects and classifies Windows, Linux, and Workstation objects. |
| **Reconciliation Engine** | Automatically relocates misplaced groups to their correct OU. |
| **Legacy Cleanup** | Detects and removes orphaned, disabled, or mismatched groups. |
| **Simulation Mode** | `WhatIf=true` enables dry-run validation without changing AD. |
| **Verbose Logging** | Detailed color-coded logging with timestamps and action categories. |

---

## 🧩 Configuration Model

All behavior is governed by a **single INI file**:  
`AD-ManageComputerGroups.ini` — serving as the **Single Source of Truth (SSOT)** for the entire pipeline.

### Example Configuration
```ini
[Config.Core]
WhatIf=true
Verbose=true
AutoCreateOUs=true
AutoCreateGlobalGroups=true
BaseReconciliationSearchDN=OU=Groups,DC=example,DC=com
```
