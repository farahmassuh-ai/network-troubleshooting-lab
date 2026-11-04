# Baseline Test Matrix

Known-good results recorded on 2026-11-04, before any faults were injected. Rerun every row after each ticket.

**Baseline result: all 12 rows pass.**

PCs and servers start TTL at 128; routers and switches start at 255. Each routing hop subtracts 1. A first ping that times out is normal: the sender is resolving ARP.

| # | Test | Expected | Actual (baseline) | Pass |
|---|---|---|---|---|
| 1 | HQ inter-VLAN: HQ-PC1 → HQ-SRV `ping 10.1.30.10` | Replies, TTL 127 | 3/4 replies, TTL 127 (first timed out: ARP) | ✅ |
| 2 | Branch inter-VLAN: BR-PC1 → `ping 10.2.20.1` and BR-GUEST1 → `ping 198.51.100.10` | Both reply | BR-PC1 → 10.2.20.1: 4/4, TTL 255. BR-GUEST1 → 198.51.100.10: 4/4, TTL 125 (an earlier run lost 2 to ARP) | ✅ |
| 3 | OSPF adjacencies: HQ-R1 `show ip ospf neighbor` | 2.2.2.2 and 3.3.3.3 FULL | 2.2.2.2 FULL/DR via Gi0/0/1, 3.3.3.3 FULL/DR via Gi0/0/2 ([output](HQ-R1_show-ip-ospf-neighbor.txt)) | ✅ |
| 4 | OSPF routes: BR-R1 `show ip route` | HQ /24s as `O`, plus `O*E2 0.0.0.0/0` | 10.1.10/20/30/99.0/24 as `O` [110/3] via 10.0.0.1; `O*E2 0.0.0.0/0` via 10.0.0.1 ([output](BR-R1_show-ip-route.txt)) | ✅ |
| 5 | Site to site: BR-PC1 → HQ-SRV `ping 10.1.30.10` | Replies, TTL 125 | 4/4 replies, TTL 125 | ✅ |
| 6 | DHCP with relay: HQ-R1 `show ip dhcp binding` | 6 leases, each in the correct subnet, from .11 up | 6 leases: 10.1.10.11-12, 10.1.20.11-12, 10.2.10.11, 10.2.20.11 ([output](HQ-R1_show-ip-dhcp-binding.txt)) | ✅ |
| 7 | Internet via PAT: HQ-PC1 and BR-PC1 → `ping 198.51.100.10`; HQ-R1 `show ip nat statistics` | Replies, TTL 125; NAT counters increase | HQ-PC1: 4/4, TTL 125. BR-PC1: 4/4, TTL 125. NAT: 4 active translations, 32 hits ([output](HQ-R1_show-ip-nat-statistics.txt)) | ✅ |
| 8 | Guest isolation: each guest → internet OK; each guest → HQ-SRV and → 10.1.99.100 fail; staff → HQ-SRV OK | As stated | All three guests reach the internet (TTL 125). HQ-GUEST1/2 → HQ-SRV and → SRV1: "Destination host unreachable" from 10.1.20.1. BR-GUEST1 → HQ-SRV and → SRV1: same, from 10.2.20.1. Staff → HQ-SRV: OK (row 1) | ✅ |
| 9 | SSH from a Staff PC to all six managed devices | Login succeeds on all six | From HQ-PC1 as `admin`: HQ-R1 (10.1.0.1), HQ-CORE (10.1.99.1), HQ-ACC1 (10.1.99.11), HQ-ACC2 (10.1.99.12), BR-R1 (10.2.10.1), BR-ACC1 (10.2.99.11) all logged in | ✅ |
| 10 | Port security: `show port-security interface` HQ-ACC1 Fa0/1, HQ-ACC2 Fa0/21 | Secure-up, sticky MAC present | Both Secure-up, 1 sticky MAC, 0 violations: HQ-PC1 00D0.BA88.0781 ([output](HQ-ACC1_show-port-security-fa0-1.txt)), HQ-SRV 00E0.B033.46C7 ([output](HQ-ACC2_show-port-security-fa0-21.txt)) | ✅ |
| 11 | STP root: HQ-CORE `show spanning-tree vlan 10` | "This bridge is the root", priority 24586 | Root, priority 24586 (24576 + VLAN 10); Gi1/0/2 and Gi1/0/3 designated, forwarding ([output](HQ-CORE_show-spanning-tree-vlan-10.txt)) | ✅ |
| 12 | Monitoring: config change or interface flap on any device | Timestamped message on SRV1, dated correctly | Messages from all six devices dated Nov 4 2026 ([screenshot](srv1-syslog-phase1.png)) | ✅ |
