# IP Address Plan

## Network Overview

The project uses the private IPv4 network `192.168.100.0/24` as the initial address space.

VLSM is used to divide the network into smaller subnets based on the number of required devices in each department.

## Subnet Allocation

| Department | Network        | CIDR | Usable Hosts | Broadcast      |
| ---------- | -------------- | ---- | -----------: | -------------- |
| Office     | 192.168.100.0  | /27  |           30 | 192.168.100.31 |
| IT         | 192.168.100.32 | /28  |           14 | 192.168.100.47 |
| Servers    | 192.168.100.48 | /29  |            6 | 192.168.100.55 |

## Office

* Network: `192.168.100.0/27`
* Gateway: `192.168.100.1`
* Usable host range: `192.168.100.2 – 192.168.100.30`
* Broadcast: `192.168.100.31`
* Planned devices: 20

## IT

* Network: `192.168.100.32/28`
* Gateway: `192.168.100.33`
* Usable host range: `192.168.100.34 – 192.168.100.46`
* Broadcast: `192.168.100.47`
* Planned devices: 10

## Servers

* Network: `192.168.100.48/29`
* Gateway: `192.168.100.49`
* Usable host range: `192.168.100.50 – 192.168.100.54`
* Broadcast: `192.168.100.55`
* Planned devices: 5

## VLSM Verification

The subnets do not overlap:

* Office: `.0 – .31`
* IT: `.32 – .47`
* Servers: `.48 – .55`

The remaining addresses from `.56` onward are reserved for future network expansion.

