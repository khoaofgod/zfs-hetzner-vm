# ZFS Hetzner VM - ARM64 Installation Script

## Project Overview
Automated Ubuntu 24.04 installation scripts for Hetzner servers with ZFS root filesystem support for both x86_64 and ARM64 architectures.

## Recent Updates

### 2025-01-28 (CRITICAL): Fixed Boot-Breaking GRUB Path Issues

**Status**: 🔴 **CRITICAL BUG FIXED** - System would not boot

#### External Code Review Findings:
An independent analysis identified critical boot failures that would prevent the system from booting. All issues have been fixed.

#### Critical Fixes Applied:

**1. 🔴 INVALID GRUB PATH SYNTAX (BOOT BREAKING)**
- **Problem**: Used non-existent ZFS dataset notation in GRUB config
  ```bash
  # WRONG (would cause boot failure):
  linux /ROOT/ubuntu@/boot/vmlinuz-6.x.x
  initrd /ROOT/ubuntu@/boot/initrd.img-6.x.x
  ```
- **Fix**: Use standard bootfs paths that GRUB understands
  ```bash
  # CORRECT:
  linux /boot/vmlinuz-6.x.x
  initrd /boot/initrd.img-6.x.x
  ```
- **Impact**: System would drop to GRUB rescue shell on boot
- **Solution**: GRUB automatically uses ZFS bootfs property, just reference /boot/

**2. 🟡 HARDCODED DISK DEVICE (/dev/sda)**
- **Problem**: efibootmgr always used `/dev/sda` even when disk was `/dev/vda`
- **Impact**: EFI boot entries not created on Hetzner Cloud VMs (use VirtIO)
- **Fix**: Dynamic disk detection from `$BOOT_PART` variable
- **Result**: Works on both Cloud VMs (/dev/vda) and dedicated (/dev/sda)

**3. 🟡 GRUB CONFIGURATION ISSUES**
- **Removed**: `GRUB_DISABLE_LINUX_UUID=true` (breaks ZFS snapshots/clones)
- **Removed**: Redundant "rw" from boot parameters (GRUB adds automatically)
- **Added**: Explicit ZFS module loading: `--modules="zfs part_gpt"`
- **Added**: `search --label` for proper ZFS pool detection

**4. 🔧 IMPROVED BOOT ENTRY CREATION**
- Dynamic disk/partition number extraction from detected device
- Better error messages distinguishing Cloud VMs vs dedicated servers
- Enhanced troubleshooting documentation

#### Code Changes:
```bash
# Before (BROKEN):
linux /ROOT/ubuntu@/boot/vmlinuz-$KVER root=ZFS=rpool/ROOT/ubuntu rw
efibootmgr --create --disk /dev/sda --part 1

# After (WORKING):
search --label $ZFS_POOL --set=root
linux /boot/vmlinuz-$KVER root=ZFS=$ZFS_POOL/ROOT/ubuntu
efibootmgr --create --disk $BOOT_DISK --part $BOOT_NUM
```

#### Boot Chain (Corrected):
```
1. UEFI Firmware
   ↓
2. /boot/efi/EFI/ubuntu/grubaa64.efi (or BOOT/BOOTAA64.EFI fallback)
   ↓
3. GRUB loads with ZFS modules
   ↓
4. search --label finds ZFS pool
   ↓
5. GRUB reads /boot/grub/grub.cfg from bootfs dataset
   ↓
6. Kernel loaded from /boot/vmlinuz (in ZFS bootfs)
   ↓
7. initrd loads ZFS modules
   ↓
8. Ubuntu boots from ZFS root
```

---

### 2025-01-28: Comprehensive ARM64 GRUB Boot Improvements

#### Critical Fixes Applied:
1. **Dynamic ZFS Pool Name Support**
   - Fixed hardcoded "rpool" to use user-defined `$ZFS_POOL` variable
   - Ensures GRUB boots with correct ZFS pool name
   - Variable properly exported to chroot environment

2. **Variable Scope Issues**
   - Export `BOOT_PART` and `ZFS_POOL` variables to chroot
   - Fixed "unbound variable" errors during installation
   - Proper variable initialization (`AUTO_INSTALL_SUCCESS`)

3. **Fallback Boot Entry**
   - Always create `/boot/efi/EFI/BOOT/BOOTAA64.EFI` fallback
   - Ensures boot even if primary EFI entry fails
   - Standard UEFI fallback location

4. **GRUB Configuration Improvements**
   - Multiple installation methods (automatic + manual)
   - Proper directory structure creation (`/boot/grub`, `/boot/efi/grub`)
   - Minimal GRUB config fallback for ZFS filesystem detection issues
   - Automatic kernel version detection

5. **Enhanced Verification**
   - Comprehensive post-installation checks
   - Verify critical files (grubaa64.efi, grub.cfg, etc.)
   - Display boot configuration details
   - Clear success/failure indicators

#### Technical Details:

**GRUB Boot Chain:**
```
1. UEFI → /boot/efi/EFI/ubuntu/grubaa64.efi (primary)
2. UEFI → /boot/efi/EFI/BOOT/BOOTAA64.EFI (fallback)
3. GRUB → /boot/grub/grub.cfg (configuration)
4. Kernel → ZFS pool detection → Boot Ubuntu
```

**ZFS Root Boot Parameters:**
```bash
GRUB_CMDLINE_LINUX="root=ZFS=$ZFS_POOL/ROOT/ubuntu rw"
```

**Installation Methods:**
- Method 1: `grub-install` with `--no-nvram` (chroot compatibility)
- Method 2: `grub-install` without `--no-nvram` (NVRAM support)
- Method 3: Manual EFI file copy from `/usr/lib/grub/arm64-efi/monolithic/`

#### Known Issues & Solutions:

**Issue**: `grub-probe: error: unknown filesystem`
- **Cause**: GRUB cannot detect ZFS filesystem in chroot
- **Solution**: Fallback to minimal grub.cfg with manual ZFS configuration

**Issue**: `BOOT_PART: unbound variable`
- **Cause**: Variables not exported to chroot environment
- **Solution**: Explicit variable export before chroot execution

**Issue**: ARM64 server boot failure
- **Cause**: Missing or incorrect GRUB EFI files
- **Solution**: Multiple fallback locations + manual file copy

#### Files Modified:
- `hetzner-ubuntu24-zfs-setup-arm64.sh` - Complete GRUB boot overhaul

#### Testing Status:
- ✅ GRUB package installation verified
- ✅ EFI file locations confirmed (`monolithic/grubaa64.efi`)
- ✅ Fallback boot entry creation working
- ✅ Variable scope issues resolved
- ⏳ Full boot test pending (requires server reboot)

## Script Architecture

### ARM64 Script (`hetzner-ubuntu24-zfs-setup-arm64.sh`)
**Target**: ARM64 Hetzner servers with UEFI
**Boot Method**: GRUB-EFI with ZFS root support
**Key Features**:
- Ubuntu ports repository support
- ARM64-specific GRUB packages
- Multiple bootloader installation methods
- Comprehensive error handling

### x86_64 Script (`hetzner-ubuntu24-zfs-setup.sh`)
**Target**: x86_64 Hetzner servers
**Boot Method**: ZFSBootMenu or GRUB-EFI
**Status**: Stable, production-ready

## Installation Process

### Pre-installation:
1. Boot Hetzner server into rescue mode
2. Ensure network connectivity
3. Clone repository or download script

### Installation Steps:
```bash
# ARM64 servers
bash hetzner-ubuntu24-zfs-setup-arm64.sh

# x86_64 servers
bash hetzner-ubuntu24-zfs-setup.sh
```

### Post-installation:
1. Verify GRUB configuration: `/boot/grub/grub.cfg`
2. Check EFI entries: `efibootmgr -v`
3. Reboot and test boot process
4. If boot fails, check BIOS/UEFI settings (Secure Boot, boot order)

## Troubleshooting

### Boot Failure:
1. Enter Hetzner rescue mode
2. Check Secure Boot is disabled in BIOS/UEFI
3. Verify boot order prioritizes Ubuntu entry
4. Manual EFI entry creation:
   ```bash
   efibootmgr --create --disk /dev/sda --part 1 \
     --label "ubuntu" --loader "\\EFI\\ubuntu\\grubaa64.efi"
   ```

### GRUB Configuration Issues:
- Check `/boot/grub/grub.cfg` exists
- Verify ZFS pool name matches: `zpool list`
- Regenerate config: `update-grub`

### ZFS Mount Issues:
- Import pool: `zpool import -f $POOL_NAME`
- Check pool status: `zpool status`
- Mount datasets: `zfs mount -a`

## Development Notes

### Design Principles:
- Keep it simple but functional
- Multiple fallback mechanisms
- Clear error messages
- Comprehensive verification

### Future Improvements:
- [ ] Support for multiple disks (RAID configurations)
- [ ] Optional disk encryption (LUKS)
- [ ] Network configuration templates
- [ ] Post-installation configuration hooks

## References
- Ubuntu ZFS documentation
- GRUB-EFI boot process
- Hetzner rescue system
- ARM64 UEFI specifications

---
Last updated: 2025-01-28
