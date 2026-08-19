# Troubleshooting 005 — TEST01 automatic checkpoint chain

## Context

I performed a final Chapter 1 audit of `TEST01` after validating Hyper-V, storage, virtual networking, static management addressing, Dynamic Memory, and host-to-guest connectivity.

## Symptom

The VM itself was healthy and reachable, but the final audit showed:

```text
AutomaticCheckpointsEnabled : True
CheckpointType              : Standard
```

`Get-VMHardDiskDrive` also showed that the active virtual disk was an `.avhdx` differencing disk instead of the base `TEST01.vhdx` file.

## Investigation

I inspected the checkpoint inventory and active disk chain before changing anything.

`Get-VMSnapshot -VMName "TEST01"` showed two checkpoints, including an automatic checkpoint and a child checkpoint.

I then inspected the active disk with `Get-VHD`. The current `.avhdx` had another `.avhdx` as its parent, confirming that TEST01 was operating through a checkpoint chain rather than directly from the base VHDX.

I did not manually delete any `.avhdx` files.

## Root Cause

Automatic checkpoints were still enabled on the temporary validation VM. Checkpoints created differencing disks, so later writes were being stored in `.avhdx` files above the original `TEST01.vhdx`.

The checkpoint feature was functioning normally; the issue was that automatic checkpoints were not the policy I wanted for this infrastructure lab.

## Resolution

I shut TEST01 down cleanly, disabled automatic checkpoints, and removed the existing checkpoints through Hyper-V:

```powershell
Set-VM `
    -Name "TEST01" `
    -AutomaticCheckpointsEnabled $false

Get-VMSnapshot -VMName "TEST01" |
Remove-VMSnapshot -Confirm:$false
```

Hyper-V merged the checkpoint data back into the parent disk chain.

## Final Verification

After cleanup I verified:

```text
TEST01 state                     Off
AutomaticCheckpointsEnabled      False
Remaining checkpoints            none
Active disk                      TEST01.vhdx
Remaining TEST01 .avhdx files    none
```

I then started TEST01 again and verified:

```text
TEST01 state                     Running
AutomaticCheckpointsEnabled      False
Remaining checkpoints            none
Active disk                      TEST01.vhdx
10.10.30.20 connectivity         4/4 successful
```

This confirmed that the checkpoint chain was merged cleanly, the VM now runs directly from the base VHDX, and the cleanup did not break guest startup or management connectivity.

## Evidence

- `screenshots/chapter-01/01-08c-test01-checkpoint-diagnosis.png`
- `screenshots/chapter-01/01-08d-test01-checkpoint-cleanup.png`
- `screenshots/chapter-01/01-08e-test01-final-post-cleanup-validation.png`

## Lesson Learned

A VM can appear healthy while still carrying an unintended checkpoint chain. A final virtualization audit should include checkpoint policy, snapshot inventory, and the active virtual-disk path. I should manage checkpoints through Hyper-V and verify the resulting VHDX chain instead of treating `.avhdx` files as ordinary files to remove manually.
