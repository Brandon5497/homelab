# Lab 01: Proxmox VE Installation and Setup

## Goal

Install Proxmox VE as a bare-metal (type 1) hypervisor on the repaired HP EliteDesk, give it a stable network identity, and bring it up to date so it's ready to host virtual machines.

## Environment

| Item | Details |
|------|---------|
| Host | HP EliteDesk 705 G4 SFF |
| Memory | 16GB DDR4 |
| Boot and VM storage | 512GB NVMe SSD |
| Hypervisor | Proxmox VE 9.2-1 |
| Network | Wired Ethernet, home LAN (192.168.1.0/24) |
| Management | Web interface on port 8006, SSH |

## Why a type 1 hypervisor

I chose Proxmox, which runs directly on the hardware, over a type 2 option such as VirtualBox on top of Windows or Linux:
- No desktop operating system using RAM before any VMs start, which matters with 16GB.
- It behaves like a real server: managed from a browser on another computer, with no monitor needed after setup.
- Snapshots, backups, containers, and virtual networking are built in, and they match how many production environments work.

## Steps

### 1. Prepare the BIOS
- Confirmed **SVM Mode** (AMD virtualization) was already enabled. It's under Advanced, then System Options.
- Confirmed USB boot was enabled in Boot Options.

### 2. Download and verify the installer
- Downloaded **Proxmox VE 9.2-1** from the official downloads page.
- The download was slow (estimates swung between an hour and days) even though a speed test on the same computer showed about 600 Mbps down, so the bottleneck was the download server and not my network.
- Checked the file with `certutil -hashfile proxmox-ve_9.2-1.iso SHA256` and compared the result to the SHA256SUM published on the page. They matched, so the slow download wasn't corrupted.

### 3. Create the bootable USB
- Wrote the ISO to a USB drive in **Rufus** using **DD Image mode**. The drive had held an Ubuntu installer before, and everything on it was erased.
- Afterward Windows showed the drive as two drives (E: and F:) and reported "the volume does not contain a recognized file system." That's expected, because the image uses Linux partitions that Windows can't read. I ejected the drive without formatting it.

### 4. Boot the installer
- Plugged the USB and the Ethernet cable into the HP, then used the **F9 boot menu** to start from the USB.
- Selected **Install Proxmox VE (Graphical)**.

### 5. Network configuration
The installer pre-filled the network settings from my router (gateway, DNS, and /24 netmask). I kept those, but changed two things:

| Setting | Value | Reasoning |
|---------|-------|-----------|
| Hostname (FQDN) | `pve1.home.arpa` | Needs a dot. `home.arpa` is the domain reserved for home networks, and the hostname is hard to rename later, so I settled it before installing. |
| IP address | `192.168.1.50/24` | The installer offered an address from the router's DHCP pool. A static address inside that pool can later be handed to another device, so I looked up the router's DHCP range and chose one outside it. |

Gateway and DNS stayed at the router's address. The installer showed the interface as connected before I continued.

### 6. Install
- Selected the 512GB NVMe drive as the target and kept the installer's default filesystem layout.
- Set a strong root password and an email address for system notifications.
- Left **automatically reboot** checked, and removed the USB when the machine restarted so it booted from the internal drive.

### 7. First login
- From another computer, opened `https://192.168.1.50:8006` and logged in as `root`. The browser warned about the self-signed certificate, which is normal on a fresh install.
- A "No valid subscription" notice appears at each login. It's only a reminder, and it doesn't limit features.
- Confirmed SSH works with `ssh root@192.168.1.50`.

### 8. Repositories and updates
Out of the box, Proxmox points at the paid enterprise repositories, so the first package refresh failed (it shows as a red "Update package database" task). To fix that, in **Node, Updates, Repositories**:
1. Disabled the **pve-enterprise** repository.
2. Disabled the **ceph enterprise** repository (I don't use Ceph on a single node).
3. Added the **No-Subscription** repository.
4. Left the two standard Debian repositories enabled, since they provide regular and security updates.

Then I ran **Refresh** (succeeded) and **Upgrade**. The upgrade finished with "Your System is up-to-date" and installed a new kernel (`7.0.14-20-pve`), which needs a reboot to take effect.

## Problems and fixes

| Problem | Cause | Fix |
|---------|-------|-----------|
| Very slow ISO download | Download server throttling, not my network | Verified my own speed, let it finish, and verified the hash |
| Windows showed the USB as two drives with an "unrecognized file system" error | DD mode wrote Linux partitions | Ignored the error, did not format, ejected the drive |
| Failed "Update package database" task | Enterprise repositories require a paid subscription | Disabled the enterprise repositories and added the no-subscription one |

## What I learned

- The difference between type 1 and type 2 hypervisors, and why a dedicated host suits a homelab.
- Verifying a download against a published checksum.
- DHCP versus static addressing, and why a server's address should sit outside the router's DHCP pool.
- Naming conventions for a private network, and why a hostname is worth choosing carefully before the install.
- Using the web interface, the node shell, and SSH to manage the same host.

## Skills practiced

Hypervisor installation, BIOS configuration, bootable media creation, checksum verification, IP planning, Linux package management (APT), remote administration over SSH and HTTPS.

## Screenshots

*Add these to a `screenshots/` folder and link them here:*
- The Proxmox web interface dashboard after first login
- The Management Network Configuration screen (hostname, IP, gateway)
- The Repositories tab showing the no-subscription repository enabled
- The completed upgrade output

## Next

[Lab 02: Ubuntu Server VM](../02-ubuntu-server-vm)
