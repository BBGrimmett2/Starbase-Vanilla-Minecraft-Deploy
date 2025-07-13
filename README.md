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

## Repository Structure

```
.
├── execution-environment/        # ansible-builder definition for the custom EE
│   └── mc-ee.yml
├── playbook.yml                  # Main Ansible playbook
├── inventories/                  # Example inventory or dynamic inventory sources
├── templates/
│   ├── start.sh.j2               # Startup script for Minecraft
│   └── minecraft.service.j2      # systemd unit file
└── vars/
    └── snow.yml                  # Optional ServiceNow credentials/vars
```

## Requirements

- Ansible Automation Platform 2.5+
- A Linux VM or physical server (e.g., provisioned via Proxmox)
- RHEL-based system (for Java package resolution)
- A valid inventory entry targeting the host
- ServiceNow integration via REST API and Ansible Spoke
- mcrcon (bundled in custom execution environment)

## Custom Execution Environment

This project uses a purpose-built execution environment hosted at:

`quay.io/bgrimmet/starbase/mc-ee`

Or build it locally:

```bash
cd execution-environment/
ansible-builder build -v3 -t mc-ee:latest -f mc-ee.yml
```

Included components:

- `ansible.posix`
- `community.proxmox`
- `mcrcon` binary
- Required system packages for Minecraft deployment

## Running the Playbook with ansible-navigator

```bash
ansible-navigator run playbook.yml \
  --eei quay.io/bgrimmet/starbase/mc-ee:latest \
  -i inventories/proxmox.yml \
  -m stdout \
  -e @vars/snow.yml \
  -e world_name=funworld \
  -e admin_players="UserOne,UserTwo" \
  -e max_players="5" \
  -e snow_rtim="RITM0011600" \
  -e justification="Just for fun!" \
  -e world_seed="MySecretSeed"
```

If using AAP and ServiceNow, these variables can be passed automatically via `extra_vars`.

## Variables

| Variable              | Description                                                | Example                |
|-----------------------|------------------------------------------------------------|------------------------|
| `world_name`          | Minecraft world name                                       | `funworld`             |
| `admin_players`       | Comma-separated Minecraft usernames with OP privileges     | `UserOne,UserTwo`      |
| `max_players`         | Number of player slots (1, 5, or 10 recommended)           | `5`                    |
| `world_seed`          | Seed for world generation                                  | `AnsibleRocks`         |
| `snow_rtim`           | (Optional) Request Item ID from ServiceNow                 | `RITM0011600`          |
| `justification`       | Justification for the request                              | `Team Testing`         |
| `servicenow_instance` | ServiceNow URL                                             | `example.snow.com`     |
| `servicenow_user`     | ServiceNow Username                                        | `dev`                  |
| `servicenow_password` | ServiceNow Password                                        | `dev`                  |