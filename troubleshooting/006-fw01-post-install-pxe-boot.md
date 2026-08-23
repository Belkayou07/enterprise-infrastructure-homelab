# Troubleshooting 006 — FW01 post-install PXE boot

## Context

I installed OPNsense 26.7 to the dedicated `FW01.vhdx` disk in the Generation 2 Hyper-V firewall VM. After the installer completed, I halted the VM, removed the OPNsense ISO from both virtual DVD drives, and verified that the VHDX remained attached before attempting the first installed-system boot.

## Symptom

The first post-install start did not reach the installed OPNsense console. Hyper-V displayed:

```text
Start PXE over IPv4
```

This showed that the Generation 2 firmware had moved on to network boot instead of successfully starting the operating system from `FW01.vhdx`.

## Investigation

I did not reinstall immediately. I first verified the virtual disk and firmware state from PowerShell.

The virtual disk was still attached correctly:

```text
VMName             FW01
ControllerType     SCSI
ControllerNumber   0
ControllerLocation 0
Path               C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
```

I also verified that Secure Boot remained disabled.

The initial firmware summary showed mixed `Drive` and `Network` entries, so I explicitly set the installed virtual hard disk as the first boot device:

```powershell
$disk = Get-VMHardDiskDrive -VMName "FW01"
Set-VMFirmware -VMName "FW01" -FirstBootDevice $disk
```

I then inspected the detailed order and confirmed:

```text
1. HardDiskDrive — SCSI 0:0 (FW01.vhdx)
2. DVD Drive — SCSI 0:1
3. WAN network adapter
4. USERS network adapter
5. SERVERS network adapter
6. MGMT network adapter
7. DVD Drive — SCSI 0:2
```

## Root Cause

The effective cause was the Hyper-V firmware boot-device priority. The installed VHDX was present and valid, but the first post-install start fell through to PXE. Explicitly setting `FW01.vhdx` as the first firmware boot device changed the result: the next start booted OPNsense successfully from disk.

I did not need to modify or reinstall the OPNsense filesystem or bootloader.

## Resolution

I explicitly placed the installed virtual hard disk first in the Generation 2 firmware boot order:

```powershell
$disk = Get-VMHardDiskDrive -VMName "FW01"
Set-VMFirmware -VMName "FW01" -FirstBootDevice $disk
```

I left the installer ISO detached and did not make any changes to the VHDX contents.

## Final Verification

I started `FW01` again after confirming the detailed boot order. OPNsense 26.7 booted from the installed VHDX, accepted the configured `root` credentials, and displayed the normal OPNsense console menu.

This verified:

```text
Installer ISO             detached
FW01.vhdx                 attached on SCSI 0:0
FW01.vhdx boot priority   first
Installed OPNsense boot   successful
Normal console menu       reached
```

The screenshot also shows the current automatic/default OPNsense interface state. I will map the four guest interfaces by MAC address before treating any interface role as authoritative.

## Evidence

- `screenshots/chapter-03/03-08-fw01-pxe-after-install.png`
- `screenshots/chapter-03/03-09-opnsense-installed-first-disk-boot.png`

## Lesson Learned

A failed first boot after installation should be treated as a boot-chain troubleshooting problem, not as an automatic reason to reinstall the operating system. I should verify the attached disk, firmware settings, and exact boot order first, change one variable at a time, and then retest before touching the installed system.
