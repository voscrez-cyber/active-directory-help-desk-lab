# HD-003 — Account Lockout and Secure Password Reset

| Field | Details |
|---|---|
| Category | Identity / Account Management |
| Priority | High |
| Status | Resolved |
| Affected account | Alex (fictional lab user) |
| Technician account | Taylor, delegated Tier 1 help desk administrator |

## Ticket Summary

A domain user was locked out after five incorrect password attempts. The help-desk technician needed to restore access without using a Domain Admin account.

## User-Reported Symptom

The user could not sign in and received a message indicating that the account was locked.

## Business Impact

The employee could not use the workstation or reach company resources. Because password resets affect a person's identity, the request also presented a security risk if the caller was impersonating the real employee.

## Investigation

1. Verified the caller's identity before touching the account.
2. Checked Active Directory for locked accounts.
3. Confirmed that the account was enabled and had reached the domain lockout threshold.
4. Reviewed the domain policy: five failed attempts, a 15-minute lockout duration, and a 15-minute observation window.
5. Used the delegated Taylor help-desk account instead of the built-in Domain Administrator.
6. Reviewed Security logs for the account-lockout and unlock activity.

## Root Cause

The user entered an incorrect password five times. This triggered the domain's account-lockout policy, which protected the account from repeated password guessing.

## Resolution

1. Verified the user's identity using the lab's support procedure.
2. Unlocked the account with the delegated help-desk account.
3. Reset the password only if the user could not remember the existing one.
4. When a reset was required, used a temporary password and required the user to change it at the next sign-in.
5. Did not add the help-desk technician to Domain Admins.

## Verification

- `Search-ADAccount -LockedOut` no longer returned the user.
- The user successfully signed in.
- If a temporary password was issued, Windows required a new private password at sign-in.
- Security logs recorded the lockout and administrative recovery actions.

## Commands Used

```powershell
# Find locked accounts
Search-ADAccount -LockedOut

# Unlock the account
Unlock-ADAccount -Identity <username>

# Reset only when required
$NewPassword = Read-Host "Enter temporary password" -AsSecureString
Set-ADAccountPassword -Identity <username> -Reset -NewPassword $NewPassword
Set-ADUser -Identity <username> -ChangePasswordAtLogon $true
```

## What I Learned

- Identity verification comes before a password reset because an attacker may pretend to be the employee.
- Lockout policy slows down password-guessing attacks.
- A help-desk technician should use delegated permissions instead of powerful Domain Admin rights.
- Good ticket notes create an audit trail showing who performed the action and why.

