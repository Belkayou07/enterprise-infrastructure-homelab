# Troubleshooting 002 — Duplicate Hyper-V Virtual Switches

## Symptom

While creating the Chapter 1 Hyper-V virtual switches, the `New-VMSwitch` commands were pasted more than once. PowerShell then showed multiple switch objects using the same friendly names:

- `vSW-MGMT`
- `vSW-SERVERS`
- `vSW-USERS`

The issue was also noticed while reviewing the configuration in Hyper-V Virtual Switch Manager.

## Cause

The switch-creation commands had been executed repeatedly instead of once per intended switch.

## Resolution

The duplicate custom switch entries were removed manually through Hyper-V Virtual Switch Manager, keeping exactly one intended switch of each type.

No VM network adapters had yet been attached to these lab switches, so the cleanup could be performed before any dependent infrastructure existed.

## Final Verification

PowerShell was used after the GUI cleanup to verify both switch counts and unique switch IDs:

```powershell
Get-VMSwitch |
    Group-Object Name |
    Sort-Object Name |
    Select-Object Name, Count

Get-VMSwitch |
    Sort-Object Name |
    Select-Object Name, SwitchType, Id
```

Final verified state:

| Switch | Count | Type |
|---|---:|---|
| `Default Switch` | 1 | Internal |
| `vSW-MGMT` | 1 | Internal |
| `vSW-SERVERS` | 1 | Private |
| `vSW-USERS` | 1 | Private |

Each final switch also reported a distinct Hyper-V switch `Id`.

## Learning Point

Creation commands should not be re-run blindly just because the previous output is no longer visible. Verify current state first, then change only what is actually missing.

The GUI was useful for visually spotting the duplicate configuration, while PowerShell provided the precise final inventory and object counts.

## Evidence

Planned evidence files:

- `screenshots/chapter-01/01-03a-duplicate-switches-detected.png`
- `screenshots/chapter-01/01-03b-virtual-switches-final.png`
- `screenshots/chapter-01/01-03c-virtual-switch-manager-gui.png`

These should only be marked committed after the actual PNG files are present in the repository.
