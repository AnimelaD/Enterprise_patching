#  Enterprise Patching Automation Framework (Ansible + AAP)

##  Overview

This project demonstrates an **enterprise-grade patching automation solution** built using Ansible and orchestrated through **Red Hat Ansible Automation Platform (AAP)**.

The solution focuses on:

*  Pre-check validation
*  Controlled patching (Linux & Windows)
*  Reboot handling
*  Post-check validation
* Workflow-based notifications

**Important:**
This repository contains **only the automation logic (roles & playbooks)**.
All execution, orchestration, credentials, and workflows are managed inside **AAP**.

---

Architecture

### 🔷 High-Level Workflow

```
Precheck
   ↓
Patch Web Tier
   ↓
Patch App Tier
   ↓
Patch DB Tier
   ↓
Patch Windows Servers
   ↓
Postcheck
   ↓
Notify
```

 Key Design Principles

* ✔ Tier-based patching (Web → App → DB → Windows)
* ✔ Dependency-aware execution
* ✔ Controlled rollout using `serial`
* ✔ Failure isolation and early exit
* ✔ Idempotent automation
* ✔ Separation of concerns (code vs orchestration)

---

 Repository Structure

```
Enterprise_patching/
│
├── playbooks/
│   ├── precheck.yml
│   ├── patch_linux.yml
│   ├── patch_windows.yml
│   ├── postcheck.yml
│   └── notify.yml
│
├── roles/
│   ├── precheck_linux/
│   ├── precheck_windows/
│   ├── patch_rhel/
│   ├── patch_windows/
│   ├── postcheck_linux/
│   └── postcheck_windows/
│
├── inventory/
│   └── hosts (placeholder only)
│
├── group_vars/
│   └── all.yml (placeholder only)
│
└── ansible.cfg
```

---

## ⚙️ Inventory & Variables

⚠️ **Note:**

* Inventory and variables in this repo are **placeholders only**
* Actual inventory is managed dynamically within **AAP**
* Credentials are securely stored in **AAP Credential Store**

### Example (Reference Only)

```ini
[web]
web1
web2

[app]
app1
app2

[db]
db1
db2

[windows]
win1
```

---

## Credential Management

All credentials are managed in **AAP**, including:

* ✔ Linux SSH credentials
* ✔ Windows WinRM credentials
* ✔ Privilege escalation (sudo)
* ✔ Cloud/API credentials (if applicable)

👉 No secrets are stored in this repository.

---

##  Workflow Orchestration (AAP)

###  Workflow Template Design

Each stage is implemented as a **Job Template** and connected via a **Workflow Template**.

| Stage         | Execution Condition |
| ------------- | ------------------- |
| Precheck      | Start               |
| Patch Web     | On Success          |
| Patch App     | On Success          |
| Patch DB      | On Success          |
| Patch Windows | On Success          |
| Postcheck     | On Success          |
| Notify        | On Failure / Always |

---

###  Convergence Strategy

* ✔ **ALL** → used for sequential execution
* ✔ **ANY** → used for failure notification aggregation

---

### Failure Handling

* Fail fast at each stage
* Stop downsteam execution
* Trigger notification node
* Preserve system stability

---

## Precheck Logic

### Linux

* Disk usage validation
* Memory usage validation
* Pending reboot detection (`needs-restarting`)

### Windows

* C: drive free space check
* Memory usage check
* Pending reboot registry checks

---

##  Patching Strategy

### Linux (RHEL)

* Uses `yum` / `dnf`
* Controlled execution with `serial`
* Reboot handled based on system requirement

```yaml
serial: 1   
```

### Windows

* Uses `win_updates`
* Controlled reboot handling via `win_reboot`
* Post-reboot validation

---

## Reboot Handling

### Linux

* Uses `needs-restarting -r`
* Conditional reboot based on return code

### Windows

* Uses `win_updates` result (`reboot_required`)
* Explicit reboot via `win_reboot`

---

## Postcheck Logic

* System availability validation
* Basic service validation
* Disk/memory sanity checks
* Ensures no "false success"

---

## Notification

* Triggered on:

  * Failure
  * Completion

* Provides:

  * Execution status
  * Timestamp
  * Target hosts

---

## Key Features

* ✔ Multi-tier patch orchestration
* ✔ Cross-platform (Linux + Windows)
* ✔ Workflow-driven execution (AAP)
* ✔ Safe rolling updates (`serial`)
* ✔ Failure containment
* ✔ Extensible and reusable design

---

##  How to Use (AAP Integration)

1. Push this repo to GitHub
2. Create a **Project** in AAP (SCM-based)
3. Create **Credentials** (Linux(RHEL)/Windows)
4. Create **Inventory** (static_
5. Create **Job Templates**:

   * precheck
   * patch_linux
   * patch_windows
   * postcheck
   * notify
6. Build **Workflow Template**
7. Execute workflow

---

##  Assumptions & Limitations

* No load balancer integration (host isolation not implemented)
* No rollback mechanism (can be extended)
* Basic postchecks (no deep application validation)
* Designed for demo / extensible to production

---

## Future Enhancements

* Load balancer integration (host isolation)
*  Automated rollback strategy
*  Application-level health checks
*  Integration with monitoring tools
*  Dynamic inventory (cloud integration)

---

## Final Thought

> This is not just a patching script — it is a **controlled, workflow-driven automation system** designed for enterprise environments.

