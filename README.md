# Starbase Vanilla Minecraft Deploy

An Ansible-powered automation playbook for deploying a **Vanilla Minecraft Server** via **Ansible Automation Platform (AAP)**, optionally integrated with **ServiceNow**.

This repository demonstrates how enterprise-grade automation tools can be used to provision and manage application environments through **governed, repeatable, and self-service workflows**—using Minecraft as a fun example.

## What This Does

- Automates the full lifecycle of a Minecraft server:
  - Validates and parses inputs
  - Installs Java and system dependencies
  - Downloads the latest Minecraft server
  - Configures world settings and RCON access
  - Sets up a `systemd` service to manage the server
  - Grants operator access via RCON
  - Updates a ServiceNow request item with results (optional)

This pattern is applicable beyond Minecraft—think Tomcat, .NET apps, training environments, or other baked application stacks.

## Requirements

- Target host:
  - RHEL-9 system
  - SSH access
- Control machine:
  - `ansible-navigator` installed
  - Internet access to pull the execution environment from Quay.io

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/BBGrimmett2/Starbase-Vanilla-Minecraft-Deploy.git
cd Starbase-Vanilla-Minecraft-Deploy
```

### 2. Prepare your inventory

Modify or create an inventory file. Example (`inventory.yml`):

```yaml
all:
  hosts:
    sb-temp01:
      ansible_user: root
      ansible_host: 192.168.1.100
```

### 3. Customize your variables

Edit `vars/snow.yml` or override via `extra_vars`. Required variables include:

snow.yml:
```yaml
servicenow_instance: example.snow.com
servicenow_user: example
servucenow_password: password
```

```yaml
admin_players: "Baconator1013, LucasIsCool"
world_name: "funbaconworld"
world_seed: "test12345"
max_players: "5"
environment: "dev"
justification: "Internal test"
snow_rtim: "RITM0011594"
```

### 4. Run with Ansible Navigator

Use the containerized execution environment hosted on Quay.io:

```bash
ansible-navigator run playbook.yml \
  --eei quay.io/bgrimmet/starbase/mc-ee \
  --inventory inventory.yml \
  --mode stdout
```

You may also pass variables with `-e` if not using a vars file:

```bash
-e '{"admin_players": "Player1, Player2", "world_name": "demo"}'
```

---

## Notes

- The playbook **removes** any existing server under `/opt/minecraft` to ensure clean deployment.
- The `minecraft.service` is managed with `systemd` and set to auto-start.
- Updates ServiceNow records

---

## Project Structure

```
.
├── inventories/
│   └── inventory.yml           # Sample inventory
├── vars/
│   └── snow.yml                # ServiceNow variable file
├── playbook.yml                # Main playbook
├── templates/                  # Service definitions, scripts, etc.
└── README.md
```

## TODO

- OS Agnostic
- Workflow with VM Deployment
