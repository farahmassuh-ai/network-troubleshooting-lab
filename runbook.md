# Runbook

First commands to run for common symptoms on this network. Version 1, written before the tickets; it grows as each ticket teaches something new.

## Method (every ticket)

1. **Define** the problem: who is affected, what fails, since when.
2. **Logs:** check SRV1 (Services → SYSLOG) for anything that changed around that time.
3. **Work up the layers:** link up? Right VLAN? Correct IP and gateway? Route to the destination? Anything filtering it?
4. **Compare** with `baseline/`: same command, same device, spot the difference.
5. **Fix** the root cause, not just the symptom.
6. **Verify** with all 12 rows of [baseline/test-matrix.md](baseline/test-matrix.md).
7. **Document** it as a ticket.

## Quick reference

| Thing | Value |
|---|---|
| Gateways | HQ: 10.1.x.1 on HQ-CORE · Branch: 10.2.x.1 on BR-R1 |
| DHCP server | HQ-R1 (10.1.0.1), reached through `ip helper-address` on each gateway |
| SRV1 (syslog + NTP) | 10.1.99.100, HQ-ACC2 Fa0/24, VLAN 99 |
| Syslog "HostName" = sender | 10.1.0.1 HQ-R1 · 10.1.99.1 HQ-CORE · 10.1.99.11 HQ-ACC1 · 10.1.99.12 HQ-ACC2 · 10.0.0.2 BR-R1 · 10.2.99.11 BR-ACC1 |
| Internet test target | 198.51.100.10 |

## Symptom → first commands

| Symptom | Where | First commands |
|---|---|---|
| **One host has no network at all** | Host | `ipconfig /all` |
| | Its access switch | `show ip interface brief` (its port up/up?) · `show port-security interface <port>` · `show mac address-table` (its MAC on that port?) |
| **Host has 169.254.x.x / no DHCP address** | Host | `ipconfig /renew` |
| | Access switch | `show vlan brief` · `show interfaces trunk` |
| | Its gateway | `show ip interface brief` · `show running-config \| include helper` |
| | HQ-R1 | `show ip dhcp binding` · `show ip dhcp pool` |
| **Host got an address in the wrong subnet** | Access switch | `show vlan brief` (which VLAN is its port in?) · `show mac address-table` |
| **Reaches its gateway but nothing beyond** | Host | `ipconfig` (gateway correct?) · `tracert <destination>` |
| | Gateway (HQ-CORE / BR-R1) | `show ip interface brief` · `show ip route` |
| **Whole VLAN can't leave its subnet** | Gateway | `show ip interface brief` (SVI or subinterface up/up?) · `show ip route` |
| **One site can't reach the other** | HQ-R1, BR-R1, HQ-CORE | `show ip ospf neighbor` · `show ip route` · `show ip protocols` · `ping` across the WAN (10.0.0.1 ↔ 10.0.0.2) |
| **Internal works, internet doesn't** | HQ-R1 | `show ip route` (default route?) · `show ip nat statistics` · `ping 203.0.113.1` |
| **Traffic blocked that should pass, or passes that shouldn't** | Gateway | `show access-lists` (watch the match counters) · `show ip interface <interface>` (which ACL, which direction?) |
| **Can't SSH to one device** | From a Staff PC | `ping <mgmt IP>` |
| | The device (console) | `show ip interface brief` (Vlan99 up?) · `show running-config \| include default-gateway\|vty\|transport\|login` · `show ip ssh` |
| **Repeated warning in the logs, users fine** | SRV1, then the device | Read the full message on SRV1 · `show logging` on the device · compare its `show` output with `baseline/` |
| **Logs missing or timestamps wrong** | The device | `show ntp status` · `show clock` · `ping 10.1.99.100` · `show running-config \| include logging\|ntp` |

## Lessons learned

- **The first ping can time out.** The sender is resolving ARP. One `.` then `!!!!` is normal; repeated failures are not.
- **Syslog can lose messages.** It's UDP: a message sent before ARP resolves is dropped and never resent. Missing logs don't prove nothing happened.
- **"Destination host unreachable" from a gateway** usually means the gateway refused to forward it (for example, an ACL), not that the destination is down.
- **`--More--`:** Packet Tracer doesn't support `terminal length 0`. Press Space to page through, or the rest of the output is lost.
- **Prompt `>` vs `#`:** some commands need privileged mode. Type `enable` first.
- **Compare, don't guess:** run the same command as the baseline file and look for the line that changed.
