# IPv6 Configuration and Connectivity Lab

## Objective
Configure IPv6 on a Cisco router and a PC, then verify connectivity.

## Network Configuration

### Router R1
- Interface: GigabitEthernet0/0
- IPv6 Address: 2001:DB8:100:10::1/64
- Link Status: up/up
- IPv6 Routing: Enabled

### PC1
- Interface: FastEthernet0
- IPv6 Address: 2001:DB8:100:10::2/64
- Default Gateway: FE80::209:7CFF:FE71:2801

## Router Configuration

```text
ipv6 unicast-routing
interface gigabitEthernet0/0
 ipv6 address 2001:DB8:100:10::1/64
 no shutdown
