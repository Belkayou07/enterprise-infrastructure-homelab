# Troubleshooting 002 — Duplicate Hyper-V Virtual Switches

## Context

During Chapter 1 of my BelkaCorp homelab, I created the custom Hyper-V switches from an elevated PowerShell session:

```powershell
New-VMSwitch -Name "vSW-USERS" -SwitchType Private
New-VMSwitch -Name "vSW-SERVERS" -SwitchType Private
New-VMSwitch -Name "vSW-MGMT" -SwitchType Internal
```

While learning and verifying the configuration, I pasted the creation commands more than once. This produced additional custom switch objects with the same friendly names.

## Observation

I noticed the duplicate configuration while checking the environment in both Hyper-V Manager / Virtual Switch Manager and PowerShell.

I intentionally use both interfaces during this lab:

- PowerShell gives me exact, scriptable state and precise verification;
- Hyper-V Manager gives me a visual model of the configuration and helped expose the unexpected switch entries.

## Cause

I executed the `New-VMSwitch` creation commands repeatedly instead of running them once and immediately verifying the resulting objects.

This was an operator workflow mistake during the manual-learning phase, not a reason to redesign the network.

## Resolution

Because I had not attached any production lab VMs to the duplicate switches yet, I removed the extra custom switch objects manually in Hyper-V Virtual Switch Manager, leaving one intended switch of each type.

I did not run a bulk cleanup script after the manual GUI remediation.

## Final Verification

I confirmed the final PowerShell inventory contained one object per switch name:

```text
Name            Count
----            -----
Default Switch      1
vSW-MGMT            1
vSW-SERVERS         1
vSW-USERS           1
```

Final types:

```text
Default Switch  Internal
vSW-MGMT        Internal
vSW-SERVERS     Private
vSW-USERS       Private
```

I also verified that each remaining switch object had a different Hyper-V `Id`.

## Lesson Learned

For infrastructure creation commands, I should use this sequence:

```text
CREATE ONCE
    |
    v
VERIFY
    |
    v
CONTINUE
```

I should not repeatedly paste a creation command just because it produced little output. I need to verify the resulting object first.

This incident also reinforced the value of using the GUI and PowerShell together: the GUI helps me understand the topology visually, while PowerShell gives me the precise state I need for validation and later automation.

## Evidence

The actual evidence files are committed in the repository:

- `screenshots/chapter-01/01-03a-duplicate-switches-detected.png`
- `screenshots/chapter-01/01-03b-virtual-switches-final.png`
- `screenshots/chapter-01/01-03c-virtual-switch-manager-gui.png`

The first screenshot records the duplicate state; the second records the cleaned PowerShell inventory; and the third records the Hyper-V Virtual Switch Manager view used during the troubleshooting process.
