# HD-002 — Department File Share Access Denied

| Field | Details |
|---|---|
| Category | Access / File Services |
| Priority | Medium |
| Status | Resolved |
| Affected resource | `\\DC01\IT` |
| Environment | `corp.voscrez.test` domain |

## Ticket Summary

A domain user could see the IT department drive or knew its network path, but Windows denied access when the user tried to open it.

## User-Reported Symptom

Windows displayed an access error when the user opened `\\DC01\IT`.

## Business Impact

The user could not reach files needed for their assigned department. Incorrect permissions can either block an employee from doing their job or accidentally expose another department's data.

## Investigation

1. Confirmed which domain account was signed in by running `whoami`.
2. Confirmed the user's department and job need before changing access.
3. Reviewed the user's Active Directory group memberships.
4. Checked the share permissions on the IT SMB share.
5. Checked the NTFS permissions on the IT folder.
6. Confirmed that access was assigned through security groups rather than directly to one user.
7. Refreshed Group Policy and the user's sign-in token after correcting group membership.

## Root Cause

The user's effective access did not match the required department role. The account either lacked the correct security-group membership or was still using an older sign-in token created before the membership change. Windows evaluates both share permissions and NTFS permissions, and the most restrictive effective result wins.

## Resolution

1. Verified the request was legitimate and matched the user's department.
2. Added the account to the appropriate department security group.
3. Removed access that was no longer required.
4. Ran `gpupdate /force`.
5. Signed the user out and back in so Windows created a new token containing the updated group memberships.

## Verification

- The authorized user could open the correct department drive.
- The mapped drive appeared with the expected letter.
- The user could not open a different department's share.
- No direct per-user permission was added.

## Commands Used

```powershell
# CLIENT01
whoami
whoami /groups
gpupdate /force

# DC01
Get-ADUser <username> -Properties MemberOf
Get-SmbShareAccess -Name IT
icacls "C:\CompanyShares\Departments\IT"
```

## What I Learned

- Seeing a drive does not automatically mean the user has permission to open it.
- A user's sign-in token is like a permission badge. After group membership changes, signing out and back in creates a fresh badge.
- Group-based access is easier to audit and maintain than giving permissions directly to individual users.
- **Least privilege** means users receive only the access required for their job.

