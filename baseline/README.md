# Baseline

Known-good state of the network, captured on 2026-11-04 after Phase 1 (SRV1 and HQ-GUEST2 added, syslog and NTP on all six managed devices) and before any faults were injected.

During each ticket, compare the broken network against these files to see exactly what changed.

## Test matrix

[test-matrix.md](test-matrix.md): all 12 rows pass.

## Show command outputs

| Device | Files |
|---|---|
| HQ-R1 | [show ip route](HQ-R1_show-ip-route.txt) · [show ip ospf neighbor](HQ-R1_show-ip-ospf-neighbor.txt) · [show ip interface brief](HQ-R1_show-ip-interface-brief.txt) · [show ip dhcp binding](HQ-R1_show-ip-dhcp-binding.txt) · [show ip nat statistics](HQ-R1_show-ip-nat-statistics.txt) |
| BR-R1 | [show ip route](BR-R1_show-ip-route.txt) · [show ip ospf neighbor](BR-R1_show-ip-ospf-neighbor.txt) · [show ip interface brief](BR-R1_show-ip-interface-brief.txt) |
| HQ-CORE | [show ip route](HQ-CORE_show-ip-route.txt) · [show ip ospf neighbor](HQ-CORE_show-ip-ospf-neighbor.txt) · [show ip interface brief](HQ-CORE_show-ip-interface-brief.txt) · [show vlan brief](HQ-CORE_show-vlan-brief.txt) · [show interfaces trunk](HQ-CORE_show-interfaces-trunk.txt) · [show spanning-tree vlan 10](HQ-CORE_show-spanning-tree-vlan-10.txt) |
| HQ-ACC1 | [show vlan brief](HQ-ACC1_show-vlan-brief.txt) · [show interfaces trunk](HQ-ACC1_show-interfaces-trunk.txt) · [show ip interface brief](HQ-ACC1_show-ip-interface-brief.txt) · [show port-security interface fa0/1](HQ-ACC1_show-port-security-fa0-1.txt) |
| HQ-ACC2 | [show vlan brief](HQ-ACC2_show-vlan-brief.txt) · [show interfaces trunk](HQ-ACC2_show-interfaces-trunk.txt) · [show ip interface brief](HQ-ACC2_show-ip-interface-brief.txt) · [show port-security interface fa0/21](HQ-ACC2_show-port-security-fa0-21.txt) |
| BR-ACC1 | [show vlan brief](BR-ACC1_show-vlan-brief.txt) · [show interfaces trunk](BR-ACC1_show-interfaces-trunk.txt) · [show ip interface brief](BR-ACC1_show-ip-interface-brief.txt) |

## Running configs

[configs/](configs/): exported running config of all six managed devices. Compared with the Project 1 configs, the only changes are the Phase 1 additions: `service timestamps log datetime msec`, `logging 10.1.99.100` and `ntp server 10.1.99.100` on every device; HQ-ACC2 Fa0/24 moved to VLAN 99 and enabled (SRV1); HQ-ACC2 Fa0/11 enabled (HQ-GUEST2); and newly learned sticky MACs.

## Monitoring

[srv1-syslog-phase1.png](srv1-syslog-phase1.png): SRV1 receiving timestamped logs from all six devices.

## Notes

- Packet Tracer doesn't support `terminal length 0`; press Space at `--More--` to get full output.
- HQ-R1's route ages display as `4294967293:...`, likely a Packet Tracer glitch after the NTP clock change. Routes are correct.
- All six devices accepted `ntp server`; no clocks were set by hand.
