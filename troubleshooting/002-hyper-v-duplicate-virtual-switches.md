# Troubleshooting 002 — Duplicate Hyper-V Virtual Switches

## Context

During Chapter 1 of the BelkaCorp homelab, the custom Hyper-V switches were created from an elevated PowerShell session:

```powershell
New-VMSwitch -Name "vSW-USERS" -SwitchType Private
New-VMSwitch -Name "vSW-SERVERS" -SwitchType Private
New-VMSwitch -Name "vSW-MGMT" -SwitchType Internal
```

While learning and verifying the configuration, the create commands were pasted more than once. This produced additional custom switch objects with the same friendly names.

## Observation

The duplicate configuration became visible while checking the environment in Hyper-V Manager / Virtual Switch Manager and PowerShell.

The user intentionally uses both interfaces during the lab:

- PowerShell provides exact, scriptable state and verification;
- Hyper-V Manager provides a visual model of the configuration and helped expose the unexpected switch entries.

## Cause

The `New-VMSwitch` creation commands were executed repeatedly instead of being run once and followed immediately by verification.

This was an operator workflow mistake during the manual-learning phase, not a reason to redesign the network.

## Resolution

Because no production lab VMs were attached to the duplicate switches yet, the extra custom switch objects were removed manually in Hyper-V Virtual Switch Manager, leaving one intended switch of each type.

No bulk cleanup script was run after the manual GUI remediation.

## Final Verification

The final PowerShell inventory confirmed one object per switch name:

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

The final inventory also displayed a different unique `Id` for each remaining switch object.

## Lesson Learned

For creation commands that change infrastructure state, use the sequence:

```text
CREATE ONCE
    |
    v
VERIFY
    |
    v
CONTINUE
```

Do not repeatedly paste a creation command simply because the command itself produced little output. Verify the resulting object first.

The incident also reinforced the value of using the GUI and PowerShell together: the GUI is useful for understanding the topology visually, while PowerShell is the authoritative way used in this project to count and inspect the resulting objects precisely.

## Evidence

Planned evidence files for this incident:

- `screenshots/chapter-01/01-03a-duplicate-switches-detected.png`
- `screenshots/chapter-01/01-03b-virtual-switches-final.png`
- `screenshots/chapter-01/01-03c-virtual-switch-manager-gui.png`

Evidence is only considered complete after the actual image files are present in the repository.
