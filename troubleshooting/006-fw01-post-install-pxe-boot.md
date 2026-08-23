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

The root cause is not yet confirmed.

The first failed boot could have been caused by the firmware not prioritizing the installed VHDX correctly, but that hypothesis still requires a controlled retry with `FW01.vhdx` explicitly first in the boot order.

If the VM still falls through to PXE after that retry, the investigation will move to the installed UEFI/bootloader state on the virtual disk rather than assuming the installation itself must be repeated.

## Resolution

Resolution is currently in progress.

As the first controlled remediation step, I explicitly placed `FW01.vhdx` first in the Generation 2 firmware boot order. I have not changed the installed disk contents or reinstalled OPNsense.

## Final Verification

Final verification is pending.

The next test is to start `FW01` again with the verified boot order. A successful OPNsense console boot from the VHDX will confirm the issue is resolved. If PXE appears again, I will continue investigating the EFI/bootloader state before considering reinstallation.

## Evidence

- `screenshots/chapter-03/03-08-fw01-pxe-after-install.png`

## Lesson Learned

A failed first boot after installation should be treated as a boot-chain troubleshooting problem, not as an automatic reason to reinstall the operating system. I should verify the attached disk, firmware settings, and exact boot order first, then change one variable at a time and retest.
