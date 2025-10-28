# ZFS Hetzner VM Installer

[![shellcheck](https://github.com/khoaofgod/zfs-hetzner-vm/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/khoaofgod/zfs-hetzner-vm/actions/workflows/shellcheck.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Automated installation scripts for Debian and Ubuntu with ZFS root filesystem on Hetzner servers (Cloud, Virtual, and Dedicated).

## 🚀 Quick Start

These scripts provide a fully automated way to install Linux distributions with ZFS as the root filesystem on Hetzner infrastructure. Perfect for servers requiring advanced storage features like snapshots, compression, and data integrity checks.

### ⚠️ Warning

**ALL DATA ON THE TARGET DISK WILL BE DESTROYED.** This is a complete system installation that will wipe your disks.

---

## 📋 Prerequisites

1. **Hetzner Server** - Cloud, VPS, or Dedicated Server
2. **Rescue Console Access** - Ability to boot into Hetzner's Linux rescue system
3. **SSH Key** - For secure access to your server

---

## 🛠️ Installation Guide

### Step 1: Boot into Rescue Mode

1. Log into your [Hetzner Robot](https://robot.hetzner.com/) or [Cloud Console](https://console.hetzner.cloud/)
2. Navigate to your server's **Rescue** menu
3. Configure rescue system:
   - Enable rescue mode
   - Select **Linux 64-bit** as the OS
   - Add your **SSH public key**
4. Click **"Mount rescue and power cycle"** to reboot into rescue mode

### Step 2: Connect via SSH

After the server reboots into rescue mode, connect via SSH:

```bash
ssh root@YOUR_SERVER_IP
```

Use the password provided by Hetzner or your SSH key for authentication.

### Step 3: Run the Installation Script

Choose the script for your desired distribution and run it directly:

#### Debian Installations

**Debian 10 (Buster)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian10-zfs-setup.sh | bash -
```

**Debian 11 (Bullseye)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian11-zfs-setup.sh | bash -
```

**Debian 12 (Bookworm)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian12-zfs-setup.sh | bash -
```

**Debian 13 (Trixie)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian13-zfs-setup.sh | bash -
```

#### Ubuntu LTS Installations

**Ubuntu 18.04 LTS (Bionic Beaver)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu18-zfs-setup.sh | bash -
```

**Ubuntu 20.04 LTS (Focal Fossa)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu20-zfs-setup.sh | bash -
```

**Ubuntu 22.04 LTS (Jammy Jellyfish)**
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu22-zfs-setup.sh | bash -
```

**Ubuntu 24.04 LTS (Noble Numbat)** ✨ Latest
```bash
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu24-zfs-setup.sh | bash -
```

### Step 4: Configure Installation

The script will prompt you for:
- **Hostname** - Your server's hostname
- **ZFS Pool Name** - Name for your ZFS pool (default: `rpool`)
- **Root Password** - Password for root user access

### Step 5: Wait for Completion

The script will:
1. Partition disks and create ZFS pools
2. Install the base system with `debootstrap`
3. Configure bootloader (GRUB/ZFSBootMenu)
4. Set up networking and SSH
5. Automatically reboot the system

After reboot, you can log in with your SSH key and the root password you set.

---

## 🔧 Advanced Usage

### Using Screen for Long-Running Installations

To protect against network interruptions during installation, use `screen`:

```bash
# Set locale and start screen session
export LC_ALL=en_US.UTF-8 && screen -S zfs

# Run your installation script inside screen
wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu24-zfs-setup.sh | bash -
```

**Screen Commands:**
- **Detach from screen:** Press `Ctrl+A`, then `D`
- **Reattach to screen:** `screen -r zfs`
- **List sessions:** `screen -ls`

### Stopping Software RAID

If your disks are part of a software RAID array, stop it before running the script:

```bash
mdadm --stop --scan
```

---

## 🎯 Features


✅ **Fully Automated Installation** - One command to complete installation
✅ **ZFS Root Filesystem** - Advanced storage with snapshots, compression, and checksumming
✅ **Multiple OS Support** - Debian 10-13 and Ubuntu 18.04-24.04 LTS
✅ **Optimized for Hetzner** - Tested on Cloud, VPS, and Dedicated Servers
✅ **Secure by Default** - SSH key authentication, no password login
✅ **EFI and BIOS Support** - Works with both modern UEFI and legacy BIOS systems
✅ **Network Configuration** - Automatic DHCP setup with systemd-networkd
✅ **Production Ready** - Minimal, stable configuration perfect for servers

---

## 📦 What Gets Installed

### Base System
- Minimal OS installation (Debian or Ubuntu)
- Linux kernel (latest stable for your distribution)
- OpenSSH server for remote access
- Essential system utilities (curl, nano, htop, net-tools)

### ZFS Components
- ZFS kernel modules (DKMS)
- ZFS utilities (zfsutils-linux)
- ZFS initramfs integration
- Automatic ZFS import and mount services

### Bootloader
- **Ubuntu 24.04**: ZFSBootMenu (modern, ZFS-native bootloader)
- **Other versions**: GRUB with ZFS support

### System Services
- systemd-networkd (networking)
- systemd-resolved (DNS resolution)
- systemd-timesyncd (NTP time sync)
- SSH daemon (remote access)

---

## 🏗️ ZFS Pool Layout

The scripts create the following ZFS structure:

```
rpool (or your chosen pool name)
├── ROOT
│   └── ubuntu (or debian)
│       ├── /tmp
│       ├── /var/tmp
│       ├── /var/log
│       └── /var/cache/apt
└── home
    └── /home
```

### Dataset Properties

| Dataset | Mountpoint | Special Properties |
|---------|------------|-------------------|
| `rpool/ROOT/ubuntu` | `/` | Boot filesystem |
| `rpool/ROOT/ubuntu/tmp` | `/tmp` | devices=off, no snapshots |
| `rpool/ROOT/ubuntu/var/tmp` | `/var/tmp` | devices=off, no snapshots |
| `rpool/ROOT/ubuntu/var/log` | `/var/log` | atime=off |
| `rpool/ROOT/ubuntu/var/cache/apt` | `/var/cache/apt` | atime=off, no snapshots |
| `rpool/home` | `/home` | Separate from OS for easy backups |

### ZFS Features Enabled

- **Compression**: LZ4 (fast, efficient)
- **ACL Type**: POSIX ACLs
- **Extended Attributes**: SA (system attribute)
- **Ashift**: 12 (4KB sectors, optimal for modern disks)

---

## 🔍 Troubleshooting

### Installation Fails with "No suitable disks found"

**Solution:** Stop any existing RAID arrays:
```bash
mdadm --stop --scan
```

### Lost Connection During Installation

**Solution:** Use `screen` to protect against network disconnects:
```bash
export LC_ALL=en_US.UTF-8 && screen -S zfs
# Run your script, then detach with Ctrl+A, D
# Reconnect later with: screen -r zfs
```

### Server Won't Boot After Installation

**Possible causes:**
1. BIOS/UEFI boot order needs adjustment
2. ZFS pool not imported on boot

**Check in rescue mode:**
```bash
zpool import -f rpool
zfs list
```

### Configuration File Conflicts (dpkg prompt)

The Ubuntu 24 script has been updated to handle configuration file conflicts automatically. If you encounter issues with older scripts, the error occurs when existing config files conflict with package updates.

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Report Issues** - Found a bug? [Open an issue](https://github.com/khoaofgod/zfs-hetzner-vm/issues)
2. **Submit Pull Requests** - Improvements and fixes are appreciated
3. **Test Scripts** - Try the scripts on different Hetzner server types
4. **Documentation** - Help improve this README

### Development

```bash
# Clone the repository
git clone https://github.com/khoaofgod/zfs-hetzner-vm.git
cd zfs-hetzner-vm

# Test scripts with shellcheck
shellcheck hetzner-*.sh
```

---

## 📝 Credits


This project is forked from [terem42/zfs-hetzner-vm](https://github.com/terem42/zfs-hetzner-vm) with improvements and updates.

**Original Author:** Andrey Prokopenko (job@terem.fr)

**Maintained by:** [khoaofgod](https://github.com/khoaofgod)

Special thanks to:
- OpenZFS project for the amazing filesystem
- Hetzner for excellent server infrastructure
- The Linux community for continuous improvements

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## ⚡ Quick Reference

| Distribution | Command |
|--------------|---------|
| Debian 10 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian10-zfs-setup.sh \| bash -` |
| Debian 11 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian11-zfs-setup.sh \| bash -` |
| Debian 12 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian12-zfs-setup.sh \| bash -` |
| Debian 13 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-debian13-zfs-setup.sh \| bash -` |
| Ubuntu 18.04 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu18-zfs-setup.sh \| bash -` |
| Ubuntu 20.04 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu20-zfs-setup.sh \| bash -` |
| Ubuntu 22.04 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu22-zfs-setup.sh \| bash -` |
| Ubuntu 24.04 | `wget -qO- https://raw.githubusercontent.com/khoaofgod/zfs-hetzner-vm/master/hetzner-ubuntu24-zfs-setup.sh \| bash -` |

---

## 🌟 Why ZFS?

ZFS offers enterprise-grade features for your servers:

- **Data Integrity** - Automatic checksumming prevents silent data corruption
- **Snapshots** - Instant, space-efficient backups
- **Compression** - Transparent LZ4 compression saves disk space
- **Easy Backups** - `zfs send/receive` for efficient remote backups
- **Copy-on-Write** - No fragmentation, consistent performance
- **RAID-Z** - Software RAID with better performance than mdadm
- **ARC Cache** - Intelligent caching for improved performance

---

## 📚 Additional Resources

- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/)
- [ZFS on Linux](https://zfsonlinux.org/)
- [Hetzner Docs](https://docs.hetzner.com/)
- [Ubuntu ZFS Guide](https://ubuntu.com/tutorials/setup-zfs-storage-pool)

---

## 💬 Support

- **Issues:** [GitHub Issues](https://github.com/khoaofgod/zfs-hetzner-vm/issues)
- **Discussions:** [GitHub Discussions](https://github.com/khoaofgod/zfs-hetzner-vm/discussions)

---

## 🔄 Changelog

### Recent Updates (Ubuntu 24.04)

- ✅ Fixed dpkg configuration file conflict handling
- ✅ Added `DEBIAN_FRONTEND=noninteractive` for automated installs
- ✅ Updated to use `--force-confold` and `--force-confdef` flags
- ✅ Improved error handling and logging
- ✅ Enhanced ZFSBootMenu integration for UEFI systems

### Coming Soon

- [ ] Support for ARM64 architecture
- [ ] Optional encryption setup
- [ ] Custom ZFS pool layouts
- [ ] Automated snapshot scheduling
- [ ] More customization options

---

<div align="center">

**Made with ❤️ for the Hetzner and ZFS communities**

[⭐ Star this repo](https://github.com/khoaofgod/zfs-hetzner-vm) • [🐛 Report Bug](https://github.com/khoaofgod/zfs-hetzner-vm/issues) • [✨ Request Feature](https://github.com/khoaofgod/zfs-hetzner-vm/issues)

</div>
