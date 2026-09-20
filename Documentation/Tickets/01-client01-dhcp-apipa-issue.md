# HD-001 — CLIENT01 DHCP/APIPA Connectivity Issue

| Field | Details |
|---|---|
| Category | Network / DHCP |
| Priority | Medium |
| Status | Resolved |
| Affected device | CLIENT01 |
| Server | DC01 (`192.168.10.10`) |
| Environment | VirtualBox internal network `LabNet` |

## Ticket Summary

CLIENT01 could not obtain a normal IPv4 address from the DHCP server. Windows assigned the computer a `169.254.x.x` address, and `ipconfig /renew` timed out.

## User-Reported Symptom

The workstation could not reach normal domain resources or resolve `dc01.corp.voscrez.test` correctly.

## Business Impact

The user could not reliably access domain services, mapped drives, or other company resources. A domain workstation needs a valid address, DNS settings, and a connection to the domain controller.

## Investigation

1. Ran `ipconfig /all` on CLIENT01.
2. Found an address beginning with `169.254`, which is an **APIPA** address.
3. Ran `ipconfig /release` and `ipconfig /renew`. The renewal timed out while trying to contact DHCP.
4. Pinged `192.168.10.10` and received replies, proving that basic communication between CLIENT01 and DC01 was possible.
5. On DC01, confirmed that the DHCP service was running.
6. Confirmed that DHCP was bound to the `LabNet` adapter at `192.168.10.10`.
7. Confirmed that the `192.168.10.0/24` DHCP scope was active and the server was authorized in Active Directory.
8. Confirmed that the Windows Firewall rules for DHCP Server traffic were enabled.
9. Verified both virtual machines were connected to the same VirtualBox internal network named `LabNet`.

## Root Cause

The exact single trigger was not conclusively identified. The evidence pointed to a temporary DHCP or virtual-adapter state problem after the virtual machines had been paused, shut down, or resumed. The server configuration itself—service, binding, scope, authorization, and firewall—was valid.

## Resolution

1. Corrected and confirmed the VirtualBox adapter connection to `LabNet`.
2. Restarted the affected DHCP/network state.
3. Released and renewed the client lease again.
4. Confirmed that CLIENT01 received a valid address from the configured scope.

## Verification

- CLIENT01 received a `192.168.10.x` address instead of `169.254.x.x`.
- The default DNS server pointed to DC01 at `192.168.10.10`.
- `ping 192.168.10.10` succeeded.
- `nslookup dc01.corp.voscrez.test` returned `192.168.10.10`.
- `Test-ComputerSecureChannel` returned `True`.

## Commands Used

```powershell
# CLIENT01
ipconfig /all
ipconfig /release
ipconfig /renew
ping 192.168.10.10
nslookup dc01.corp.voscrez.test
Test-ComputerSecureChannel

# DC01
Get-Service DHCPServer
Get-DhcpServerv4Binding
Get-DhcpServerv4Scope
Get-DhcpServerInDC
Get-NetFirewallRule -DisplayGroup "DHCP Server"
```

## What I Learned

- An address beginning with `169.254` means Windows gave itself an emergency address because it did not receive a DHCP lease.
- A successful ping proves basic network communication, but it does **not** prove that DHCP is working.
- DHCP troubleshooting should be done in layers: adapter, network, service, binding, scope, authorization, firewall, and finally the client lease.

