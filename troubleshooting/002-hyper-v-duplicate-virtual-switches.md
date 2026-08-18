# Troubleshooting 002 — Duplicate Hyper-V Virtual Switches

## Context

During Chapter 1 of my BelkaCorp homelab, I created the custom Hyper-V switches from an elevated PowerShell session:

```powershell
New-VMSwitch -Name "vSW-USERS" -SwitchType Private
New-VMSwitch -Name "vSW-SERVERS" -SwitchType Private
New-VMSwitch -Name "vSW-MGMT" -SwitchType Internal
```

I was using both PowerShell and Hyper-V Virtual Switch Manager while learning and validating the platform.

## Symptom

I accidentally pasted the creation commands more than once. PowerShell then showed multiple custom switch objects using the same friendly names, and I also noticed the unexpected entries while reviewing the configuration in Virtual Switch Manager.

The affected names were:

- `vSW-MGMT`
- `vSW-SERVERS`
- `vSW-USERS`

## Investigation

I compared the GUI view with the exact PowerShell inventory instead of continuing to create more switches.

PowerShell was useful for counting the objects and checking their unique Hyper-V IDs, while Virtual Switch Manager helped me understand the topology visually and identify which entries needed to be removed.

## Root Cause

I executed the `New-VMSwitch` creation commands repeatedly instead of creating each switch once and immediately verifying the resulting state.

This was an operator workflow mistake during the manual-learning phase, not a problem with the network design itself.

## Resolution

Because I had not attached any production lab VMs to the duplicate switches yet, I removed the extra custom switch objects manually in Hyper-V Virtual Switch Manager and kept exactly one intended switch of each type.

I did not run a bulk cleanup script after the manual GUI remediation.

## Final Verification

I verified the final switch counts with PowerShell:

```text
Name            Count
----            -----
Default Switch      1
vSW-MGMT            1
vSW-SERVERS         1
vSW-USERS           1
```

I also verified the final switch types:

```text
Default Switch  Internal
vSW-MGMT        Internal
vSW-SERVERS     Private
vSW-USERS       Private
```

Each remaining switch object also reported a different Hyper-V `Id`.

## Evidence

The actual evidence is committed in the repository:

- `screenshots/chapter-01/01-03a-duplicate-switches-detected.png`
- `screenshots/chapter-01/01-03b-virtual-switches-final.png`
- `screenshots/chapter-01/01-03c-virtual-switch-manager-gui.png`

The first screenshot records the duplicate state, the second records the cleaned PowerShell inventory, and the third records the Virtual Switch Manager view used during the troubleshooting process.

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

I should not repeatedly paste a creation command just because it produced little output. This incident also reinforced the value of using the GUI and PowerShell together: the GUI helps me understand the topology, while PowerShell gives me precise state for validation and later automation.