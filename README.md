# Ansible RHEL Patching Framework

A production-oriented Ansible framework for safely patching Red Hat Enterprise Linux systems through prechecks, controlled deployment batches, conditional reboot handling, post-patch validation, and documented safety controls.

## Project Purpose

Manual server patching can be slow, inconsistent, and difficult to audit. Applying updates without prechecks or staged deployment can also introduce unnecessary production risk.

This project demonstrates a reusable automation approach for patching RHEL systems while maintaining operational control, validation, and clear failure visibility.

## Key Capabilities

* Validates supported operating systems and package managers
* Checks root-filesystem free space before patching
* Supports YUM and DNF
* Supports full or security-only patching
* Supports package exclusions
* Processes systems in controlled batches
* Detects whether a reboot is required
* Prevents reboots during Ansible check mode
* Validates critical services after patching
* Provides a separate read-only validation playbook
* Uses fictional inventory data suitable for public demonstration

## Technologies

* Red Hat Enterprise Linux
* Ansible
* YAML
* Bash and Linux utilities
* YUM and DNF
* systemd
* SSH

## Repository Structure

```text
ansible-rhel-patching/
├── ansible.cfg
├── inventories/
│   └── lab/
│       ├── group_vars/
│       │   └── all.yml
│       └── hosts.yml
├── playbooks/
│   ├── patch.yml
│   └── validate.yml
├── roles/
│   └── rhel_patching/
│       ├── defaults/
│       │   └── main.yml
│       ├── handlers/
│       └── tasks/
│           └── main.yml
├── docs/
├── diagrams/
├── sample-output/
├── scripts/
├── .gitignore
├── LICENSE
└── README.md
```

## Workflow

```text
Inventory selection
        |
        v
Connectivity validation
        |
        v
Operating-system and package-manager validation
        |
        v
Root-filesystem free-space check
        |
        v
Controlled package update
        |
        v
Reboot-requirement detection
        |
        v
Conditional reboot
        |
        v
Critical-service validation
        |
        v
Completion reporting
```

## Safety Controls

The framework includes the following operational safeguards:

1. **Supported-platform validation**
   The role stops if the target is not a Red Hat family system using YUM or DNF.

2. **Free-space validation**
   Patching stops when the root filesystem has less than 1 GB available.

3. **Controlled batches**
   The default lab configuration patches 25 percent of the targeted systems at a time.

4. **Failure threshold**
   The playbook does not advance normally to additional batches when a host in the active batch fails.

5. **Conditional reboot**
   A system is rebooted only when the operating system reports that a reboot is required and rebooting has been authorized.

6. **Check-mode protection**
   The reboot task is explicitly prevented from running during Ansible check mode.

7. **Post-patch validation**
   Critical services are checked after patching and reboot processing.

8. **SSH host-key checking**
   Host-key verification remains enabled to protect against connecting to an unexpected system.

## Requirements

The Ansible control node should have:

* Linux or Windows Subsystem for Linux
* Ansible Core
* SSH connectivity to the managed systems
* Key-based SSH authentication
* A remote automation account with authorized sudo access
* RHEL lab systems using YUM or DNF

Ansible is not intended to run natively as a control node from standard Windows PowerShell. Windows users should use WSL, a Linux virtual machine, or another Linux-based control node.

## Lab Inventory

The included inventory uses fictional documentation addresses:

```yaml
rhel-lab-01:
  ansible_host: 192.0.2.10

rhel-lab-02:
  ansible_host: 192.0.2.11
```

These addresses are examples only and will not connect to real systems.

Never commit production inventories, passwords, private keys, tokens, internal domains, or real infrastructure identifiers to a public repository.

## Validate the Project

Display the inventory:

```bash
ansible-inventory --graph
```

Check playbook syntax:

```bash
ansible-playbook playbooks/validate.yml --syntax-check
```

```bash
ansible-playbook playbooks/patch.yml --syntax-check
```

List the targeted hosts without connecting:

```bash
ansible rhel_servers --list-hosts
```

## Read-Only Validation

Run the validation playbook:

```bash
ansible-playbook playbooks/validate.yml
```

The validation playbook collects:

* Ansible connectivity status
* Operating-system version
* Running kernel
* System uptime
* Root-filesystem utilization
* Critical-service status

## Check Mode

Preview the patching workflow:

```bash
ansible-playbook playbooks/patch.yml --check
```

Check mode is useful for reviewing expected actions, but it does not guarantee that every module can predict all changes perfectly. Results should still be reviewed before an authorized patching window.

## Limit the First Run

Start with one authorized test system:

```bash
ansible-playbook playbooks/patch.yml --limit rhel-lab-01
```

Only use this after replacing the sample inventory with an approved lab inventory and validating SSH and sudo access.

## Security-Only Patching

In `inventories/lab/group_vars/all.yml`, set:

```yaml
patch_security_only: true
```

Return it to `false` when the approved scope requires all available package updates.

## Excluding Packages

Example:

```yaml
patch_exclude_packages:
  - kernel*
  - database-package*
```

Package exclusions should be based on documented technical requirements and approved change scope. Excluding security-sensitive packages indefinitely can leave systems vulnerable.

## Reboot Control

To prevent automated reboots:

```yaml
patch_reboot_if_required: false
```

When disabled, reboot-required results must be handled through a separate approved change.

## Production Considerations

Before adapting this framework for production:

* Use separate development, test, and production inventories
* Store sensitive variables in Ansible Vault or an approved secret manager
* Validate backups and recovery procedures
* Coordinate with application owners
* Confirm console or out-of-band access
* Define maintenance windows
* Test application health checks
* Confirm load-balancer and monitoring procedures
* Document rollback criteria
* Capture patching and validation evidence
* Use change-control approval
* Begin with a small canary group

## Rollback Considerations

Package patching does not provide one universal rollback method. Recovery depends on the affected package, application, operating-system version, storage design, and available backups.

A production rollback plan may include:

* Booting a previously installed kernel
* Downgrading an affected package
* Restoring application configuration
* Restoring a validated snapshot or backup
* Removing a failed node from service
* Returning traffic to previously validated nodes

Rollback procedures must be tested before the production change.

## Limitations

* The included hosts are fictional and cannot be contacted.
* Application-specific health checks are not included.
* Backup products and hypervisor snapshots are outside this repository.
* Rollback is documented as an operational process rather than automatically performed.
* The `needs-restarting` utility must exist on the managed host for automatic reboot detection.
* Production use requires environment-specific testing and authorization.

## Portfolio and Privacy Notice

This repository is a sanitized lab implementation based on real-world infrastructure patterns. All hostnames, IP addresses, accounts, organizations, and environment values are fictional.

No employer, customer, proprietary, credential, or production information is included.

## Author

Solomon Nsah
Senior Linux Infrastructure Engineer