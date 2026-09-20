# HD-004 — RSAT Installation Failure on CLIENT01

| Field | Details |
|---|---|
| Category | Software / Windows Administration |
| Priority | Low |
| Status | Resolved |
| Affected device | CLIENT01 |
| Feature | RSAT: Active Directory Domain Services and LDS Tools |

## Ticket Summary

The Active Directory administration tools were missing from CLIENT01. The first installation attempt stalled and then returned: `Cannot create a file when that file already exists.`

## User-Reported Symptom

The delegated help-desk account could not open Active Directory Users and Computers from the client workstation.

## Business Impact

The technician could not perform approved user-management tasks from the help-desk workstation and would otherwise need to sign in directly to the domain controller.

## Investigation

1. Opened PowerShell as an administrator.
2. Checked the capability state and found `State : NotPresent`.
3. Attempted installation with `Add-WindowsCapability`.
4. The process ran for an extended time and later returned a file-already-exists servicing error.
5. Restarted CLIENT01 to clear any pending Windows servicing operation.
6. Retried the installation from an elevated session using DISM/Windows Optional Features.

## Root Cause

RSAT was not installed, and Windows' optional-feature servicing state encountered a conflict during the first attempt. The exact conflicting file was not identified, so the cause was documented without guessing beyond the available evidence.

## Resolution

1. Restarted CLIENT01.
2. Signed in with an account allowed to install Windows capabilities.
3. Opened an elevated PowerShell window.
4. Installed the RSAT Active Directory capability using DISM.
5. Restarted the workstation after the installation completed.

## Verification

- The capability state changed from `NotPresent` to `Installed`.
- `dsa.msc` opened Active Directory Users and Computers.
- The delegated help-desk account could perform allowed tasks such as resetting passwords and unlocking users.
- The account could not perform unrestricted Domain Admin tasks.

## Commands Used

```powershell
# Check the current state
Get-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0

# Install the capability from an elevated terminal
DISM /Online /Add-Capability /CapabilityName:Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0

# Open Active Directory Users and Computers
dsa.msc
```

## What I Learned

- RSAT lets administrators manage Windows Server roles remotely instead of working directly on the server.
- `NotPresent` means the optional capability is available but not installed.
- Administrative elevation and a clean Windows servicing state matter when adding capabilities.
- A restart is a valid troubleshooting step when an installation is stuck in a pending or conflicting state, but the result must still be verified afterward.

