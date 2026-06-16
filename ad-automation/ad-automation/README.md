# Active Directory Automation

A set of small, single-purpose Ansible playbooks to stand up Active Directory
and manage it day-to-day. Each playbook does exactly one job, so it maps 1:1 to
an AWX job template (a button in the web UI). Nothing is hardcoded: values are
prompted at the CLI, or supplied by an AWX survey form.

## What's here

| Playbook              | What it does                                   | Run as        |
|-----------------------|------------------------------------------------|---------------|
| `promote-domain.yml`  | Create a new forest / first DC (run once)      | local admin   |
| `manage-ou.yml`       | Create one Organizational Unit                 | domain admin  |
| `manage-user.yml`     | Create one user in a chosen OU                 | domain admin  |
| `manage-group.yml`    | Create one security/distribution group         | domain admin  |
| `manage-admin.yml`    | Add an existing user to a privileged group     | domain admin  |
| `join-computer.yml`   | Join a Windows machine to the domain           | local admin   |
| `list-ous.yml`        | List existing OUs (a read-only helper)         | domain admin  |

## Two design ideas worth knowing

1. **The domain is discovered, not typed.** A domain controller only ever
   serves its own domain, so the DC-side playbooks ask the DC for its own
   domain (`Get-ADDomain`) and build every Distinguished Name from that. The
   only place a domain name is configured is `group_vars/all.yml`, used solely
   by `join-computer.yml` (a not-yet-joined client can't self-discover).

2. **The same playbooks work at the CLI and in AWX.** They use `vars_prompt`,
   which prompts interactively at the command line. In AWX, a survey supplies
   those same variables as extra-vars, and Ansible skips a prompt whenever its
   variable is already defined. So one set of files serves both worlds — the
   only rule is that survey variable names must match the prompt names (they do).

## Prerequisites

- A Linux control node (or WSL) with Ansible and `pywinrm` installed.
- Collections: `ansible-galaxy collection install -r requirements.yml`
- WinRM reachable on the targets; a static IP on the DC.
- For joins: each client's DNS must point at the DC.

## Running at the CLI

Edit `inventory/hosts.yml` with your host IPs. Provide credentials via an
ansible-vault file or `--extra-vars`. Then, for example:

```
ansible-playbook playbooks/promote-domain.yml
ansible-playbook playbooks/manage-ou.yml
ansible-playbook playbooks/manage-user.yml
```

Each will prompt for what it needs.

## The GUI

See `awx/README-awx.md` for wiring these into AWX as a friendly web console.

## Secrets

No passwords live in this repo. At the CLI, keep them in an `ansible-vault`
encrypted file. In AWX, attach Machine Credentials to job templates — AWX
stores them encrypted and injects them at run time.
