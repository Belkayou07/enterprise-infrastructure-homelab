# Troubleshooting 004 — TEST01 Dynamic Memory Minimum Too Low

## Context

During Chapter 1 of my BelkaCorp homelab, I used `TEST01` as a temporary Ubuntu Server VM to validate Hyper-V CPU, memory, storage, networking, and reboot persistence before deploying the real infrastructure roles.

I originally created TEST01 with:

```text
Generation     2
vCPU           2
Startup RAM    2048 MB
Guest OS       Ubuntu 26.04 LTS
```

## Symptom

During guest runtime validation, `free -h` showed only about 465 MiB of total memory available inside Ubuntu even though I had configured 2048 MB of startup RAM.

At the same time, the other runtime checks were healthy:

```text
CPU            2 vCPU
Virtual disk   20 GB
IPv4           10.10.30.20/24
Host ping      successful
```

The unexpected memory value was therefore investigated separately instead of treating the whole VM as broken.

## Investigation

I inspected the VM's Hyper-V memory configuration from the Windows host with `Get-VMMemory` and the live assigned/demand values with `Get-VM`.

The observed configuration was:

```text
DynamicMemoryEnabled  True
StartupMB             2048
MinimumMB              512
MaximumMB          1048576
Buffer                   20
```

While the VM was running, Hyper-V reported:

```text
AssignedMB  880
DemandMB    457
```

This explained why the guest no longer had approximately 2 GiB available after startup. Dynamic Memory was enabled, and Hyper-V had reclaimed unused memory toward the configured 512 MB minimum.

## Root Cause

The VM was not suffering from a failed Ubuntu installation or missing physical host memory.

The root cause was an unsuitable Dynamic Memory configuration for this Linux guest:

- startup RAM was 2048 MB;
- minimum RAM was only 512 MB;
- the maximum was left at an unnecessarily large default value.

Because the guest demand was low, Hyper-V correctly reduced the assigned memory, but the configured minimum was lower than I wanted for a deliberately sized Ubuntu Server VM.

## Resolution

I shut TEST01 down cleanly from Ubuntu and kept Dynamic Memory enabled, but changed the limits to an intentional range:

```text
Startup RAM   2048 MB
Minimum RAM   1536 MB
Maximum RAM   4096 MB
Buffer          20%
```

I applied the change with:

```powershell
Set-VMMemory `
    -VMName "TEST01" `
    -DynamicMemoryEnabled $true `
    -StartupBytes 2GB `
    -MinimumBytes 1536MB `
    -MaximumBytes 4GB `
    -Buffer 20
```

I then verified the configuration before booting the VM again.

## Final Verification

After a cold boot, Hyper-V reported:

```text
TEST01 state    Running
AssignedMB      1536
DemandMB         522
StartupMB       2048
MinimumMB       1536
MaximumMB       4096
```

Inside Ubuntu I also verified:

```text
CPU             2
eth0            10.10.30.20/24
Connected route 10.10.30.0/24
Host ping       4/4 successful
```

The guest-visible memory value is not expected to equal the Hyper-V assigned value byte-for-byte because the guest OS and virtualization stack reserve some memory. The important verification is that Hyper-V now respects the deliberate 1536 MB minimum instead of reclaiming memory toward the previous 512 MB floor.

The static TEST01 management address and route also survived the shutdown/start cycle, so the same validation confirmed that my Netplan configuration is persistent.

## Evidence

The actual evidence files are committed in the repository:

- `screenshots/chapter-01/01-07a-test-vm-runtime-low-memory-detected.png`
- `screenshots/chapter-01/01-07b-test-vm-dynamic-memory-diagnosis.png`
- `screenshots/chapter-01/01-07c-test-vm-dynamic-memory-corrected.png`
- `screenshots/chapter-01/01-07d-test-vm-post-memory-fix-validation.png`
- `screenshots/chapter-01/01-07e-test-vm-dynamic-memory-runtime.png`

Together they show the initial low guest-memory observation, the Hyper-V Dynamic Memory diagnosis, the corrected limits, and the final guest/host runtime verification.

## Lesson Learned

Startup RAM and currently assigned RAM are not the same thing when Hyper-V Dynamic Memory is enabled.

My troubleshooting sequence for this type of mismatch is:

```text
OBSERVE GUEST MEMORY
        |
        v
CHECK HYPER-V MEMORY MODE
        |
        v
COMPARE STARTUP / MIN / MAX
        |
        v
CHECK ASSIGNED VS DEMAND
        |
        v
SIZE LIMITS INTENTIONALLY
        |
        v
COLD BOOT + VERIFY AGAIN
```

I should not assume that a low guest memory reading means the VM creation settings were ignored. I need to inspect the hypervisor's active resource-management policy first.