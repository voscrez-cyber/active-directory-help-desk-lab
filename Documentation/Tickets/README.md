# Help Desk Ticket Case Studies

These case studies document real troubleshooting exercises completed in the `corp.voscrez.test` Active Directory lab. They are written like professional help-desk notes: what the user saw, what was investigated, what caused the issue, how it was fixed, and how the fix was verified.

> All users, company names, passwords, IP addresses, and events in this repository are fictional and exist only inside an isolated home lab.

## Ticket Index

| Ticket | Issue | Main skills demonstrated |
|---|---|---|
| [HD-001](01-client01-dhcp-apipa-issue.md) | CLIENT01 received an APIPA address and could not contact DHCP | TCP/IP, DHCP, PowerShell, layered troubleshooting |
| [HD-002](02-department-share-access-denied.md) | A user could not access the correct department share | AD groups, NTFS/share permissions, least privilege |
| [HD-003](03-account-lockout-password-reset.md) | A user account was locked after repeated failed sign-ins | Identity verification, delegated administration, auditing |
| [HD-004](04-rsat-installation-failure.md) | RSAT installation failed on the help-desk workstation | Windows capabilities, DISM, elevation, verification |

## Troubleshooting Method Used

Every ticket follows the same simple process:

1. **Understand the symptom** — What does the user see?
2. **Check the basics** — Is the device connected, and is the correct user signed in?
3. **Test one layer at a time** — Network, DNS, service, permissions, or policy.
4. **Fix the smallest confirmed problem** — Avoid changing unrelated settings.
5. **Verify the result** — Prove the original problem is gone.
6. **Document the lesson** — Record what should be checked next time.

