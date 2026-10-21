# Lab 02: Ubuntu Server VM

## Goal

Create my first virtual machine on the Proxmox host: install Ubuntu Server, manage it over SSH, and use snapshots to experiment safely.

## Environment

| Item | Details |
|------|---------|
| Hypervisor | Proxmox VE 9.2 on `pve1` |
| Guest OS | Ubuntu Server 24.04 LTS |
| VM ID / name | 100 / `ubuntu-test` |
| CPU | 2 cores |
| Memory | 4 GB |
| Disk | 32 GB virtual disk (VirtIO SCSI, on LVM-thin storage) |
| Network | VirtIO network card on bridge `vmbr0`, address from the home router via DHCP |

I sized it small on purpose. The host has 16GB of RAM, and I want room to run other VMs, including the Windows Server lab, alongside it.

## Steps

### 1. Get the installer ISO onto Proxmox
- In the Proxmox web interface, I opened the `local` storage, then **ISO Images**, then **Download from URL**, so the host downloads the file directly and I don't have to copy it from my PC.
- My first attempt failed with *"Parameter verification failed: wrong file extension."* I had copied the link behind the website's Download button, which points at a redirect page and doesn't end in `.iso`. Proxmox builds the file name from the end of the URL and rejects anything that isn't an ISO. I fixed it by using the direct link from Ubuntu's release directory (`releases.ubuntu.com`), where the file name ends in `.iso`.

### 2. Create the VM
Used **Create VM** and set:
- **OS:** the Ubuntu Server ISO
- **System:** QEMU Guest Agent enabled
- **Disk:** 32 GB
- **CPU:** 2 cores
- **Memory:** 4096 MB
- **Network:** default bridge `vmbr0`

### 3. Install Ubuntu Server
Started the VM and ran the installer from the Proxmox console.
- **Network:** The virtual network card (`ens18`, a VirtIO device) picked up an address from the router by DHCP, which confirmed the VM could reach the network.
- **Storage:** Chose **Use an entire disk** with the default **LVM** layout and left disk encryption off. The installer can only see the VM's virtual 32GB disk, so this never touched the host's real drive. By default it made the root volume about 15GB and left the other half of the disk unallocated, which I noted so I could extend it (see "Problems and fixes").
- **Profile:** Set the server name, a username, and a password.
- **SSH:** Selected **Install OpenSSH server** and left password authentication on. I skipped importing an SSH key for now and plan to set up key-based login as a separate exercise.
- Skipped the optional "featured snaps" so I can install software by hand.

### 4. First boot and cleanup
- At the end of the install, the console printed *"Failed unmounting /cdrom"* and asked me to remove the installation medium. This is a known harmless quirk of virtual installs, so I pressed Enter and the VM rebooted into the installed system.
- In the VM's **Hardware** tab, set the CD/DVD drive to **Do not use any media** so the VM can't boot back into the installer.

<p align="center">
  <img src="screenshots/hardware.jpg" alt="VM hardware tab" width="450">
</p>

### 5. Connect over SSH
From my Windows PC, `ssh <username>@192.168.1.220` worked on the first try. I accepted the host fingerprint and logged in with the password I set.

### 6. Update and install the guest agent
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install qemu-guest-agent -y
```
Running `sudo systemctl enable --now qemu-guest-agent` printed a message that the unit has no `[Install]` section. This is expected on Ubuntu, since the service is started automatically when the VM exposes the agent channel, so there was nothing to fix. The VM's Summary page in Proxmox now shows its IP addresses, which is only possible when the agent is talking to the host, so that confirmed it works.

<p align="center">
  <img src="screenshots/summary.jpg" alt="VM summary" width="450">
</p>

### 7. Snapshots
Took a snapshot of the clean install, changed the system, and rolled back. The Proxmox task history shows the **VM 100 - Rollback** task. The aim was to see how a snapshot gives me a safe way to try things and undo them.

<p align="center">
  <img src="screenshots/snapshot.jpg" alt="Snapshot of current state" width="450">
</p>

## Problems and fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| ISO download failed with "wrong file extension" | I copied a redirect link, not a direct link to the `.iso` file | Used the direct download link from Ubuntu's release directory |
| Root volume only about 15GB of the 32GB disk | The installer's default LVM layout doesn't use the whole disk | Noted the unused space and extended the logical volume (or left it for later with `lvextend` and `resize2fs`) |
| Wasn't sure how to edit the volume in the installer menu | Selecting `/` in the file system summary only offers *close* and *unmount* | The edit option is on the logical volume under **Used Devices** |
| "Failed unmounting /cdrom" at the end of the install | Harmless virtual-CD quirk | Pressed Enter and removed the ISO from the VM afterward |

## What I learned

- Creating a VM and choosing sensible CPU, memory, disk, and network settings for a host with limited RAM.
- Why a thin or default allocation can leave disk space unused, and how LVM separates volumes from the physical disk.
- Managing a headless server entirely over SSH.
- What the QEMU guest agent does: it lets the host see the VM's IP address and shut it down cleanly.
- Using snapshots and rollback to experiment without risk.
- Reading an error message closely. "Wrong file extension" pointed straight at the problem.

## Skills practiced

VM provisioning in Proxmox, Linux installation, LVM storage basics, SSH remote administration, APT package management, snapshots and rollback, troubleshooting from error messages.
