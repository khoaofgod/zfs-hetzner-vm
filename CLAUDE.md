# ZFS Hetzner VM - ARM64 Installation Script

## Project Overview
Automated Ubuntu 24.04 installation scripts for Hetzner servers with ZFS root filesystem support for both x86_64 and ARM64 architectures.

## Recent Updates

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
