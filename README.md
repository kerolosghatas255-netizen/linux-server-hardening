# Linux Server Hardening Lab

## Overview

I built this project to practice Linux server administration and security hardening in a lab environment.

The first part of the project was done on an Ubuntu VM hosted on a separate VMware ESXi server. I managed the VM remotely from a Windows machine over SSH.

Later, I continued the project on VMware Workstation on my local PC. I exported the configuration from the ESXi lab, verified the backup with SHA-256, restored it to the new VM, and tested the security controls again before continuing.

## Lab Environment

The project used two environments.

The first was an Ubuntu VM running on VMware ESXi. This is where I did the initial SSH hardening, firewall setup, Fail2ban configuration, Nginx setup, backups, and logging tests.

The second environment is an Ubuntu VM running on VMware Workstation. I used it to restore the previous work, validate the configuration again, and continue with monitoring and restore testing.

Main environment details:

- Ubuntu 24.04 LTS
- VMware ESXi
- VMware Workstation
- Windows workstation for administration
- OpenSSH for remote access

## What I Configured

I started by securing SSH access.

I created a separate admin user, configured ED25519 key authentication, disabled SSH password login, and disabled direct root SSH login.

For network protection, I enabled UFW with a default deny policy for incoming traffic. SSH and HTTP are only allowed from the lab network.

I configured Fail2ban for SSH and used nftables as the ban action.

Nginx was installed as a simple web service. I used it to test firewall rules, service monitoring, access logs, error logs, and log rotation.

Automatic security updates were enabled with unattended-upgrades.

I also wrote a backup script that creates timestamped archives of the important configuration files. The backups run automatically using a systemd timer and the script keeps the latest seven copies.

## Custom Scripts

I wrote three Bash scripts while working on the lab.

### security-status

`security-status` checks the main security settings on the server.

It checks:

- SSH key authentication
- root SSH login
- password authentication
- keyboard-interactive authentication
- UFW
- Fail2ban
- Nginx
- automatic security updates
- backup scheduling
- latest backup
- listening TCP ports

Run it with:

```bash
sudo security-status
```

### server-health

`server-health` is used for basic server monitoring.

It checks:

- uptime
- CPU load
- memory usage
- root disk usage
- SSH
- Nginx
- Fail2ban
- backup timer
- latest backup age
- listening TCP ports

The script returns one of three overall states:

```text
HEALTHY
WARNING
CRITICAL
```

I tested it by stopping Nginx. The script detected that Nginx was down and changed the overall status to `CRITICAL`.

After starting Nginx again, the status returned to `HEALTHY`.

### hardening-backup

`hardening-backup` creates a compressed backup of the important configuration files and scripts used in the project.

It creates timestamped archives, uses restricted file permissions, and keeps the newest seven backups.

## Validation and Testing

I tried to test each part of the setup instead of only checking the configuration files.

Some of the tests I performed:

- logged in using the ED25519 SSH key
- confirmed that password-only SSH login is rejected
- confirmed that root SSH login is disabled
- enabled UFW and tested SSH again before closing the existing session
- restricted SSH and HTTP to the lab network
- tested a Fail2ban ban and confirmed that the IP appeared in nftables
- removed the test ban and confirmed that the jail returned to normal
- tested Nginx locally and from the Windows workstation
- generated a controlled HTTP 404 and found it in the Nginx access log
- checked the Nginx logrotate configuration
- checked the unattended-upgrades configuration and timers
- tested the backup script manually
- tested the systemd backup timer
- powered the VM off and confirmed that the persistent timer ran the missed backup after startup
- stopped Nginx and confirmed that `server-health` reported `CRITICAL`
- started Nginx again and confirmed that the status returned to `HEALTHY`
- extracted a backup to a temporary directory and compared restored files with the live files

## Backup and Restore

Backups are stored in:

```text
/var/backups/hardening
```

The backup contains the main files needed to rebuild the lab, including:

- SSH hardening configuration
- authorized SSH public keys
- Fail2ban configuration
- UFW configuration
- Nginx configuration
- automatic update configuration
- `security-status`
- `server-health`
- `hardening-backup`
- systemd backup service
- systemd backup timer

The backup files are created with restricted permissions.

I also tested the restore process without changing the live system.

The latest backup was extracted into a temporary directory and several important files were compared with their active versions using `cmp`.

The checks returned:

```text
SSH HARDENING RESTORE = MATCH
SECURITY-STATUS RESTORE = MATCH
SERVER-HEALTH RESTORE = MATCH
BACKUP SCRIPT RESTORE = MATCH
```

## Moving the Lab

While working on the ESXi VM, I found that the original Ubuntu environment was running from a live ISO instead of a persistent installation.

Before changing anything, I exported the hardening configuration and copied the recovery archive to my Windows PC.

I calculated SHA-256 hashes on both Linux and Windows and confirmed that the files matched.

Ubuntu was then installed to the virtual disk and I confirmed that the system booted from an ext4 filesystem instead of the live overlay.

Later, when I moved the project to VMware Workstation, I copied the recovery bundle to the new VM and checked the SHA-256 hash again before restoring anything.

I restored the project step by step and tested SSH, UFW, Fail2ban, Nginx, backups, and monitoring again on the new environment.

## Current Security Setup

The current SSH hardening configuration is:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```

UFW uses a default deny policy for incoming traffic.

In the current VMware Workstation lab:

```text
22/tcp  - allowed from the VMware NAT lab network
80/tcp  - allowed from the VMware NAT lab network
```

Fail2ban protects SSH and uses nftables for temporary bans.

## Logging

During the project I worked with:

- `journalctl`
- SSH authentication logs
- Nginx access logs
- Nginx error logs
- logrotate

For one test, I requested a page that did not exist and received an HTTP 404 response. I then checked the Nginx access log and found the same request there.

## Repository Structure

```text
linux-server-hardening/
├── configs/
│   ├── apt/
│   ├── fail2ban/
│   ├── nginx/
│   ├── ssh/
│   └── systemd/
├── docs/
│   ├── BACKUP_TIMER.txt
│   ├── FAIL2BAN_STATUS.txt
│   ├── SECURITY_STATUS.txt
│   ├── SERVER_HEALTH.txt
│   └── UFW_STATUS.txt
├── scripts/
│   ├── hardening-backup
│   ├── security-status
│   └── server-health
├── screenshots/
└── README.md
```

## Tools Used

- Ubuntu Linux
- OpenSSH
- Bash
- UFW
- Fail2ban
- nftables
- Nginx
- systemd
- unattended-upgrades
- journalctl
- logrotate
- VMware ESXi
- VMware Workstation

## Notes

This repository does not contain private SSH keys, passwords, live backup archives, or other credentials.

The main goal of the project was to configure the server, test the controls, deliberately create a few safe failure cases, and verify that I could detect and recover from them.
