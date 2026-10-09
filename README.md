# Homelab

A home lab built on a repurposed HP EliteDesk, used to practice virtualization, Linux, Windows Server, and Active Directory administration for IT support and, later, security work.

## Hardware

| Component | Details |
|-----------|---------|
| Machine | HP EliteDesk 705 G4 SFF |
| Memory | 16GB DDR4 |
| Storage | 512GB NVMe SSD |
| Network | Wired Ethernet, static IP on a home LAN |

## Software

- **Hypervisor:** Proxmox VE 9.2 (bare metal)
- **Guest operating systems:** Ubuntu Server 24.04 LTS, Windows Server (planned), Windows 10/11 (planned)

## Labs

| # | Lab | Status | Skills practiced |
|---|-----|--------|------------------|
| 00 | [Hardware troubleshooting: no-POST diagnosis](./00-hardware-troubleshooting) | Complete | Beep and LED codes, RAM isolation testing, BIOS settings |
| 01 | [Proxmox VE installation and setup](./01-proxmox-setup) | Complete | Bare-metal install, static IP planning, repositories and updates |
| 02 | [Ubuntu Server VM](./02-ubuntu-server-vm) | Complete | VM creation, LVM storage, SSH, snapshots, QEMU guest agent |
| 03 | [Active Directory lab](./03-active-directory) | In progress | Domain controller, DNS, users and groups, Group Policy, domain join |

## What I'm learning

- Installing and managing a type 1 hypervisor
- Planning IP addressing and avoiding DHCP conflicts
- Using snapshots to experiment safely and roll back
- Troubleshooting hardware faults methodically
- Documenting systems clearly

## Repository layout

```
homelab/
├── 00-hardware-troubleshooting/
├── 01-proxmox-setup/
├── 02-ubuntu-server-vm/
├── 03-active-directory/
└── README.md
```

Each lab folder has its own README with the goal, steps, problems encountered, and lessons learned, plus screenshots.

***Note**: Lab write-ups were drafted with help from an AI assistant (Claude) and edited by me. All hardware work, installs, and troubleshooting were done by me on my own equipment.
