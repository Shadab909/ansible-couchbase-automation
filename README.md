# Ansible Couchbase Cluster Automation (Prerequisites + Bootstrap)

This repository contains a **production-oriented Ansible setup** to prepare Linux servers for running **Couchbase** by enforcing kernel, OS, and runtime prerequisites in an **idempotent and repeatable** way.

The project is designed and implemented from scratch to reflect **real-world DevOps / SRE practices**, not lab shortcuts.

---

## 🎯 Project Goals

- Prepare Linux servers for Couchbase deployment
- Enforce Couchbase OS prerequisites consistently
- Ensure **idempotency** (safe to re-run)
- Support **multiple Linux distributions**
- Follow **Ansible best practices** (roles, inventories, handlers, guards)

---

## 🏗️ Architecture Overview
Control Node (WSL - Rocky Linux)
|
| SSH (key-based)
|
+------------------------------+
| Managed Nodes (VMs) |
| |
| cb1, cb2, cb3, cb4 |
| Rocky Linux / Debian |
+------------------------------+

- **Control Node**: WSL (Linux)
- **Managed Nodes**: VirtualBox VMs
- **Access**: Passwordless SSH
- **Privilege Escalation**: sudo (NOPASSWD)

---

## 📂 Repository Structure

ansible/
├── ansible.cfg
├── inventories/
│ └── dev/
│ └── hosts.ini
├── playbooks/
│ ├── install_python_playbook.yml
│ └── couchbase_prerequisite_playbook.yml
├── roles/
│ ├── install_python/
│ │ └── tasks/
│ │ ├── main.yml
│ │ ├── linux.yml
│ │ └── windows.yml
│ └── couchbase_prerequisite/
│ ├── tasks/
│ │ ├── main.yml
│ │ ├── swappiness.yml
│ │ ├── numa.yml
│ │ └── thp.yml
│ ├── files/
│ │ └── disable_thp.service
│ └── handlers/
│ └── main.yml
└── .gitignore

---

## 🔧 Implemented Roles

### 1️⃣ install_python

**Purpose**
- Ensure Python runtime availability on managed hosts
- Required for Ansible module execution

**Highlights**
- OS-specific task separation
- Uses `dnf`, `apt`, and Windows modules
- Conditional execution using Ansible facts

---

### 2️⃣ couchbase_prerequisite

Implements Couchbase-recommended OS tuning.

#### Swappiness
- Enforced using `ansible.posix.sysctl`
- Persistent across reboots
- Fully idempotent

#### NUMA
- Runtime detection via `/proc/cmdline`
- Debian-based systems:
  - Appends `numa=off` safely to GRUB
  - Updates GRUB only when required
- RedHat-based systems:
  - Uses `grubby` to manage kernel arguments
- Reboot triggered **only when configuration drift is detected**

#### Transparent Huge Pages (THP)
- Runtime validation using `/sys/kernel/mm/transparent_hugepage`
- Managed via a systemd oneshot service that:
  - Disables THP
  - Disables THP defrag
- Ensures correct ordering:
  - Copy unit file
  - Reload systemd daemon
  - Enable and start service
- No unnecessary reboots

---

## 🔁 Idempotency Strategy

Every change follows this workflow:

1. Read current system state
2. Apply changes only when required
3. Reload services or reboot **only if needed**

Re-running playbooks results in:
- No repeated changes
- No unnecessary reboots
- Stable and predictable outcomes

---

## ▶️ Usage

### Install Python on all nodes
```bash
ansible-playbook playbooks/install_python_playbook.yml
```

### Apply Couchbase prerequisites
```bash 
ansible-playbook playbooks/couchbase_prerequisite_playbook.yml
```
