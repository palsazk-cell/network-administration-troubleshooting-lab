# Network Administration & Troubleshooting Lab

## Project Overview

This project demonstrates practical IPv4 addressing, subnetting, Ethernet switching, VLANs, trunking, static and default routing, routing troubleshooting, DHCP, DNS, and NAT overload in Cisco Packet Tracer.

The topology evolved over several phases. Sections describing early tests are labelled as historical where their settings were later changed. The **Current Routing and NAT State** section records the final routing design used for the NAT test.

> **Simulation note:** `198.51.100.0/24` and `203.0.113.0/24` are documentation/example address ranges. In this lab they represent a simulated external network, not the real Internet.

## 1. Topology Overview

```text
                  Office VLAN 10: 192.168.100.0/27
       PC1 / PC2 / SW2 Office PC (192.168.100.4)
                         |
                 SW1 ===== SW2
                    trunk Fa0/8
                   /         \
          R1 Gi0/0              R2 Gi0/0
       192.168.100.1          192.168.100.5
              |                     |
       R1 routes between            +---- R2 Gi0/1: 192.168.200.1/24
       VLANs 10, 20, 30                   Remote PC: 192.168.200.2/24

       R1 Gi0/2: 192.168.100.33/28 (IT VLAN 20)
       R1 Gi0/1: 192.168.100.49/29 (Servers VLAN 30)

       R2 Gi0/2: 203.0.113.2/30
                     |
       R3 Gi0/0: 203.0.113.1/30
       R3 Gi0/1: 198.51.100.1/24
                     |
       Internet-PC: 198.51.100.10/24
```

R1 uses separate physical interfaces for VLANs 10, 20, and 30. Inter-VLAN routing therefore does not use router-on-a-stick. SW1 and SW2 carry VLAN traffic between switches over an 802.1Q trunk.

## 2. IPv4 Addressing Plan

### 2.1 Office Network — VLAN 10

| Item | Address / setting |
|---|---|
| Network | `192.168.100.0/27` |
| Subnet mask | `255.255.255.224` |
| Usable host range | `192.168.100.1`–`192.168.100.30` |
| Broadcast | `192.168.100.31` |
| R1 gateway | `192.168.100.1` |
| R2 inside interface | `192.168.100.5` |
| SW2 Office PC (static assignment recorded earlier) | `192.168.100.4/27`, gateway `192.168.100.1` |
| DHCP pool start | `192.168.100.10` |

Early tests used PC1 `192.168.100.2` and PC2 `192.168.100.3`. After DHCP was configured, the recorded leases were PC1 `192.168.100.10` and PC2 `192.168.100.11`. Host assignments can change if DHCP leases or Packet Tracer device settings are changed; check `ipconfig /all` on the endpoint when an exact device-to-IP mapping matters.

### 2.2 IT Network — VLAN 20

| Item | Address / setting |
|---|---|
| Network | `192.168.100.32/28` |
| Subnet mask | `255.255.255.240` |
| Usable host range | `192.168.100.33`–`192.168.100.46` |
| R1 gateway | `192.168.100.33` |
| PC3 (static assignment recorded) | `192.168.100.34` |
| Broadcast | `192.168.100.47` |

### 2.3 Servers Network — VLAN 30

| Item | Address / setting |
|---|---|
| Network | `192.168.100.48/29` |
| Subnet mask | `255.255.255.248` |
| Usable host range | `192.168.100.49`–`192.168.100.54` |
| R1 gateway | `192.168.100.49` |
| Server | `192.168.100.50` |
| Broadcast | `192.168.100.55` |

### 2.4 Remote Network

| Item | Address / setting |
|---|---|
| Network | `192.168.200.0/24` |
| Subnet mask | `255.255.255.0` |
| R2 Gi0/1 gateway | `192.168.200.1` |
| Remote PC | `192.168.200.2` |

### 2.5 Simulated External Network

| Device / interface | Address / setting |
|---|---|
| R2 Gi0/2 | `203.0.113.2/30` |
| R3 Gi0/0 | `203.0.113.1/30` |
| R3 Gi0/1 | `198.51.100.1/24` |
| Internet-PC | `198.51.100.10/24`, gateway `198.51.100.1` |

## 3. Router Interfaces

### R1

| Interface | IP address | Purpose | Recorded status |
|---|---|---|---|
| GigabitEthernet0/0 | `192.168.100.1/27` | Office VLAN 10 | up/up |
| GigabitEthernet0/1 | `192.168.100.49/29` | Servers VLAN 30 | up/up |
| GigabitEthernet0/2 | `192.168.100.33/28` | IT VLAN 20 | up/up |

### R2

| Interface | IP address | Purpose | NAT role | Recorded status |
|---|---|---|---|---|
| GigabitEthernet0/0 | `192.168.100.5/27` | Office-side link | inside | up/up |
| GigabitEthernet0/1 | `192.168.200.1/24` | Remote network | inside | up/up |
| GigabitEthernet0/2 | `203.0.113.2/30` | Link to R3 | outside | up/up |

### R3

| Interface | IP address | Purpose | Recorded status |
|---|---|---|---|
| GigabitEthernet0/0 | `203.0.113.1/30` | Link to R2 | up/up |
| GigabitEthernet0/1 | `198.51.100.1/24` | Simulated external LAN | up/up |

## 4. Switching, VLANs, and Trunking

### 4.1 VLANs

| VLAN | Name | Network | Gateway |
|---:|---|---|---|
| 10 | Office | `192.168.100.0/27` | `192.168.100.1` |
| 20 | IT | `192.168.100.32/28` | `192.168.100.33` |
| 30 | Servers | `192.168.100.48/29` | `192.168.100.49` |

The VLANs were created on the switches with the following configuration:

```text
vlan 10
 name office
vlan 20
 name IT
vlan 30
 name servers
```

### 4.2 SW1 Access-Port Assignments

| Switch port | Connected device | VLAN | Mode |
|---|---|---:|---|
| Fa0/1 | R1 Gi0/0 | 10 | Access |
| Fa0/2 | PC1 | 10 | Access |
| Fa0/3 | PC2 | 10 | Access |
| Fa0/4 | Server | 30 | Access |
| Fa0/5 | R1 Gi0/1 | 30 | Access |
| Fa0/6 | PC3 | 20 | Access |
| Fa0/7 | R1 Gi0/2 | 20 | Access |

### 4.3 SW1–SW2 Trunk

- Connection: `SW1 Fa0/8 ↔ SW2 Fa0/8`
- Both ports were configured as 802.1Q trunk ports.
- VLANs 10, 20, and 30 were created on SW2 and carried over the trunk.
- The recorded `show interfaces trunk` output showed Fa0/8 trunking with 802.1Q and active VLANs `1,10,20,30`.
- A PC on SW2 Fa0/1 used VLAN 10 with static address `192.168.100.4/27` and gateway `192.168.100.1`.
- R2 Gi0/0 was connected to SW2 Fa0/2 in VLAN 10, using `192.168.100.5/27`.

The switch MAC address tables were observed during testing and are dynamic snapshots, not permanent assignments. Earlier snapshots recorded these entries:

| Device | MAC address | Switch port |
|---|---|---|
| R1 Gi0/0 | `0009.7c71.2801` | Fa0/1 |
| PC1 | `00e0.f912.c167` | Fa0/2 |
| PC2 | `000a.f3d6.55a7` | Fa0/3 |
| Server | `0001.636c.412e` | Fa0/4 |
| R1 Gi0/1 | `0009.7c71.2802` | Fa0/5 |
| PC3 (later snapshot) | `0001.43de.8262` | Fa0/6 |
| R1 Gi0/2 (later snapshot) | `0009.7c71.2803` | Fa0/7 |

A switch learns a source MAC address from incoming Ethernet frames and associates it with the port where the frame arrived. The table can change as devices send traffic or entries age out.

## 5. Inter-VLAN Routing and Initial Connectivity

R1 provides Layer 3 connectivity between the three VLANs through its separate physical interfaces:

```text
VLAN 10 → R1 Gi0/0 → 192.168.100.1
VLAN 20 → R1 Gi0/2 → 192.168.100.33
VLAN 30 → R1 Gi0/1 → 192.168.100.49
```

The initial routing table on R1 showed these directly connected networks:

| Network | Type | Interface |
|---|---|---|
| `192.168.100.0/27` | Connected | Gi0/0 |
| `192.168.100.32/28` | Connected | Gi0/2 |
| `192.168.100.48/29` | Connected | Gi0/1 |

Connectivity tests recorded during the initial and VLAN phases included:

| Test | Recorded result |
|---|---|
| PC2 `192.168.100.3` → Office gateway `192.168.100.1` | 4/4 replies, 0% loss |
| PC2 `192.168.100.3` → PC1 `192.168.100.2` | 4/4 replies, 0% loss |
| Server `192.168.100.50` → gateway `192.168.100.49` | 4/4 replies, 0% loss |
| PC3 `192.168.100.34` → gateway `192.168.100.33` | 4/4 replies, 0% loss |
| PC3 → Office gateway `192.168.100.1` | 4/4 replies, 0% loss |
| PC3 → PC1 `192.168.100.2` | 4/4 replies, 0% loss |
| PC3 → Server `192.168.100.50` | 4/4 replies, 0% loss |
| PC on SW2 `192.168.100.4` → Office gateway `192.168.100.1` | 4/4 replies, 0% loss |
| PC on SW2 `192.168.100.4` → PC1 `192.168.100.2` | 4/4 replies, 0% loss |

One early Server-to-PC1 test returned 3/4 replies. Later cross-VLAN testing returned 4/4 replies. The initial single-packet loss was consistent with first-time ARP resolution in the simulation, but it should not be treated as a persistent fault.

## 6. Static Routing to the Remote Network

### Topology

```text
Remote PC (192.168.200.2) → R2 (192.168.200.1)
R2 Office-side IP: 192.168.100.5 ↔ R1 Office IP: 192.168.100.1
```

On R1, the route to the Remote network was configured as:

```text
ip route 192.168.200.0 255.255.255.0 192.168.100.5
```

The recorded routing-table entry was:

```text
S 192.168.200.0/24 [1/0] via 192.168.100.5
```

A traceroute from the Remote PC to `192.168.100.1` showed the path through `192.168.200.1` and then `192.168.100.1`. The Remote PC successfully pinged `192.168.100.1` with 4/4 replies and 0% packet loss after the route was restored.

## 7. Default Routing — Historical Phase and Current Change

During the earlier routing phase, R2 used R1 as its default next hop:

```text
ip route 0.0.0.0 0.0.0.0 192.168.100.1
```

That was the route used for the earlier default-routing lesson and its verification. **It is no longer the current default route.** During Phase 8, R2's default route was changed to the simulated ISP router R3:

```text
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

The old default route via `192.168.100.1` was removed with:

```text
no ip route 0.0.0.0 0.0.0.0 192.168.100.1
```

For the current Phase 8 topology, R2's default route is therefore via `203.0.113.1`.

## 8. Routing Troubleshooting Exercise

A routing failure was intentionally introduced by removing R1's route to `192.168.200.0/24`. The Remote PC then failed to ping `192.168.100.1` (4 sent, 0 received). Troubleshooting consisted of checking `show ip route`, identifying the missing route, restoring it, and retesting connectivity.

The route was restored on R1 with:

```text
ip route 192.168.200.0 255.255.255.0 192.168.100.5
```

After restoration, the routing table again showed:

```text
S 192.168.200.0/24 [1/0] via 192.168.100.5
```

The final ping from the Remote PC to `192.168.100.1` succeeded with 4/4 replies and 0% loss.

Troubleshooting workflow:

```text
Connectivity test
       ↓
Check routing table
       ↓
Identify missing route
       ↓
Restore route
       ↓
Verify connectivity
```

## 9. DHCP Configuration and Verification

### DHCP Pool on the Server

| Setting | Value |
|---|---|
| Pool name | `Office` |
| Client network | `192.168.100.0/27` |
| Subnet mask | `255.255.255.224` |
| Default gateway | `192.168.100.1` |
| DNS server | `192.168.100.50` |
| Starting address | `192.168.100.10` |
| Maximum users | `20` |

The Server uses a static address of `192.168.100.50/29`, with gateway `192.168.100.49`. DHCP Service was enabled on the Server.

Because DHCP clients are in VLAN 10 and the DHCP server is in VLAN 30, R1 relays DHCP requests from its Office interface:

```text
interface gigabitEthernet 0/0
 ip helper-address 192.168.100.50
```

The recorded lease assignments after DHCP was enabled were:

| Device | Recorded DHCP address | Gateway |
|---|---|---|
| PC1 | `192.168.100.10` | `192.168.100.1` |
| PC2 | `192.168.100.11` | `192.168.100.1` |

Both Office clients successfully pinged the Server at `192.168.100.50`, with 4/4 replies and 0% loss. `ipconfig /all` showed the DHCP server and DNS server as `192.168.100.50`.

## 10. DNS Configuration and Verification

- DNS server: `192.168.100.50`
- DNS service: enabled on the Server
- A record: `server.network.local` → `192.168.100.50`
- DHCP distributes `192.168.100.50` as the DNS server to Office clients.

The following command was tested from an Office client:

```text
ping server.network.local
```

The hostname resolved to `192.168.100.50` and the test returned 4/4 replies with 0% packet loss. This verifies DNS name resolution and connectivity to the Server in the simulated lab.

## 11. Phase 8 — NAT and Simulated External Connectivity

### Objective

Configure NAT Overload (PAT) on R2 so clients in the networks permitted by ACL 1 can use R2's outside interface address when accessing the simulated external network.

### 11.1 Current Routes Used for Phase 8

On R1, the route to the simulated external LAN was added:

```text
ip route 198.51.100.0 255.255.255.0 192.168.100.5
```

R1 also retains the static route to the Remote network:

```text
ip route 192.168.200.0 255.255.255.0 192.168.100.5
```

On R2, the current default route points to R3:

```text
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

On R3, the recorded return route to the `192.168.100.0/24` address block is:

```text
ip route 192.168.100.0 255.255.255.0 203.0.113.2
```

### 11.2 NAT Inside and Outside Interfaces on R2

```text
interface gigabitEthernet 0/0
 ip nat inside
exit

interface gigabitEthernet 0/1
 ip nat inside
exit

interface gigabitEthernet 0/2
 ip nat outside
exit
```

| Interface | Address | NAT role |
|---|---|---|
| Gi0/0 | `192.168.100.5/27` | inside |
| Gi0/1 | `192.168.200.1/24` | inside |
| Gi0/2 | `203.0.113.2/30` | outside |

### 11.3 ACL and PAT Rule

ACL 1 permits the Office subnet and the Remote subnet:

```text
access-list 1 permit 192.168.100.0 0.0.0.31
access-list 1 permit 192.168.200.0 0.0.0.255
```

NAT Overload uses the IP address assigned to R2's outside interface:

```text
ip nat inside source list 1 interface GigabitEthernet0/2 overload
```

The ACL does **not** currently include the IT subnet `192.168.100.32/28` or Servers subnet `192.168.100.48/29`. NAT success has not been established for those two VLANs, so this documentation does not claim that all VLANs have verified external access.

### 11.4 Verification

The Internet-PC was configured with:

```text
IP address:      198.51.100.10
Subnet mask:     255.255.255.0
Default gateway: 198.51.100.1
```

R3 Gi0/1 used `198.51.100.1/24`, and R3 Gi0/0 used `203.0.113.1/30`. Both routers' relevant interfaces were reported up/up. The Internet-PC successfully pinged its gateway `198.51.100.1` with 4/4 replies.

An Office client then pinged the simulated external host:

```text
ping 198.51.100.10
```

The repeat test returned 4/4 replies with 0% packet loss. The first test had timeouts followed by a reply; later repeat testing succeeded consistently.

The R2 NAT translation table showed entries with these values:

| NAT field | Observed value |
|---|---|
| Inside local | `192.168.100.11` |
| Inside global | `203.0.113.2` |
| Outside local | `198.51.100.10` |
| Outside global | `198.51.100.10` |
| Protocol | ICMP |

This is direct evidence that traffic from the host using `192.168.100.11` was translated to `203.0.113.2` for the ping test. The earlier DHCP record assigns `.11` to PC2; because the console prompt in the NAT test did not itself identify the Packet Tracer device, the safest description is “Office client with IP `192.168.100.11`” unless the endpoint-to-IP mapping is reconfirmed using `ipconfig /all`.

### Phase 8 Result

NAT Overload was configured on R2 and verified with both a successful ping to `198.51.100.10` and entries in `show ip nat translations`. The verified test covers an Office client using `192.168.100.11` and the simulated external network. It is not a test of real Internet access, and external connectivity for the IT and Servers subnets remains unverified.

## 12. Useful Verification Commands

### On Cisco routers

```text
show ip interface brief
show ip route
show running-config
show ip nat translations
show ip nat statistics
```

### On Cisco switches

```text
show vlan brief
show interfaces trunk
show mac address-table
```

### On Packet Tracer PCs

```text
ipconfig /all
ping <destination-ip-or-hostname>
tracert <destination-ip>
```

## 13. Key Concepts Demonstrated

- IPv4 addressing, subnet masks, network ranges, and broadcast addresses.
- VLAN segmentation and access-port assignment.
- 802.1Q trunking between switches.
- Inter-VLAN routing through separate physical router interfaces.
- Static routes and default routes.
- Routing failure diagnosis using the routing table and connectivity tests.
- DHCP address allocation and DHCP relay with `ip helper-address`.
- DNS A records and hostname resolution.
- NAT Overload (PAT), ACL-based source selection, and verification through NAT translations.
- Distinguishing a successful simulated connectivity test from verified real Internet connectivity.
