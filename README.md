# Day-04-Router-on-a-Stick
# CCNA Lab Day 4 – Router-on-a-Stick (Inter-VLAN Routing)

## Overview

This lab introduces Router-on-a-Stick (ROAS), a common method used to enable communication between multiple VLANs using a single router interface. In previous labs, devices in different VLANs could not communicate because VLANs are separate Layer 2 networks. In this lab, a router performs Layer 3 routing between VLANs.

## Objectives

After completing this lab, you will be able to:

- Configure VLANs on a switch
- Configure a trunk link between a switch and a router
- Create router subinterfaces
- Configure 802.1Q encapsulation
- Configure default gateways
- Enable communication between VLANs
- Troubleshoot Router-on-a-Stick issues

## Topology

```text
                    R1
                     |
                   G0/0
                     |
                  Fa0/1
                    SW1
       _____________|_____________
      |       |       |          |
    PC1     PC2     PC3        PC4

   VLAN10  VLAN20  VLAN30    VLAN40
```

## Devices Used

- 1 Cisco Router
- 1 Cisco Switch
- 4 PCs
- Copper Straight-Through Cables

## Physical Connections

| Device | Interface | Connected To |
|----------|----------|----------|
| R1 | G0/0 | SW1 Fa0/1 |
| PC1 | Fa0 | SW1 Fa0/2 |
| PC2 | Fa0 | SW1 Fa0/3 |
| PC3 | Fa0 | SW1 Fa0/4 |
| PC4 | Fa0 | SW1 Fa0/5 |

## IP Addressing

### VLAN 10 (HR)

| Device | IP Address | Default Gateway |
|----------|----------|----------|
| PC1 | 192.168.10.10/24 | 192.168.10.1 |

### VLAN 20 (Finance)

| Device | IP Address | Default Gateway |
|----------|----------|----------|
| PC2 | 192.168.20.10/24 | 192.168.20.1 |

### VLAN 30 (Sales)

| Device | IP Address | Default Gateway |
|----------|----------|----------|
| PC3 | 192.168.30.10/24 | 192.168.30.1 |

### VLAN 40 (IT)

| Device | IP Address | Default Gateway |
|----------|----------|----------|
| PC4 | 192.168.40.10/24 | 192.168.40.1 |

## Configuration Steps

### Step 1: Create VLANs on the Switch

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name FINANCE

vlan 30
name SALES

vlan 40
name IT
```

### Step 2: Configure Access Ports

```bash
interface fa0/2
switchport mode access
switchport access vlan 10

interface fa0/3
switchport mode access
switchport access vlan 20

interface fa0/4
switchport mode access
switchport access vlan 30

interface fa0/5
switchport mode access
switchport access vlan 40
```

### Step 3: Configure Trunk Port Toward Router

```bash
interface fa0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40
```

Verify:

```bash
show interfaces trunk
```

### Step 4: Enable Router Interface

```bash
enable
configure terminal

interface g0/0
no shutdown
```

Do not assign an IP address to the physical interface.

### Step 5: Configure VLAN 10 Subinterface

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

### Step 6: Configure VLAN 20 Subinterface

```bash
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

### Step 7: Configure VLAN 30 Subinterface

```bash
interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```

### Step 8: Configure VLAN 40 Subinterface

```bash
interface g0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
```

## Verification Commands

### Verify Router Interfaces

```bash
show ip interface brief
```

Expected Result:

```text
GigabitEthernet0/0.10
GigabitEthernet0/0.20
GigabitEthernet0/0.30
GigabitEthernet0/0.40
```

All interfaces should be up.

### Verify Routing Table

```bash
show ip route
```

Expected Result:

```text
C 192.168.10.0/24
C 192.168.20.0/24
C 192.168.30.0/24
C 192.168.40.0/24
```

### Verify Trunk Configuration

```bash
show interfaces trunk
```

## Connectivity Testing

### Test VLAN 10 to VLAN 20

```bash
ping 192.168.20.10
```

Expected Result:

Success

### Test VLAN 10 to VLAN 30

```bash
ping 192.168.30.10
```

Expected Result:

Success

### Test VLAN 10 to VLAN 40

```bash
ping 192.168.40.10
```

Expected Result:

Success

## How Router-on-a-Stick Works

Router-on-a-Stick uses a single physical router interface and multiple logical subinterfaces.

Example:

```text
G0/0.10 → VLAN 10 Gateway
G0/0.20 → VLAN 20 Gateway
G0/0.30 → VLAN 30 Gateway
G0/0.40 → VLAN 40 Gateway
```

Each subinterface acts as the default gateway for its VLAN.

When a PC sends traffic to another VLAN:

1. The traffic is sent to the default gateway.
2. The switch forwards the frame through the trunk.
3. The router receives the tagged frame.
4. The router performs Layer 3 routing.
5. The traffic is forwarded to the destination VLAN.

## Data Flow Example

### PC1 (VLAN 10) to PC3 (VLAN 30)

```text
PC1
 |
SW1
 |
Trunk
 |
R1

Routing Decision

R1
 |
Trunk
 |
SW1
 |
PC3
```

The router routes traffic between the VLANs.

## Troubleshooting Scenarios

### Scenario 1: Missing Encapsulation

```bash
interface g0/0.20
no encapsulation dot1Q 20
```

Expected Result:

VLAN 20 loses connectivity.

Reason:

The router can no longer identify VLAN 20 traffic.

### Scenario 2: Incorrect Default Gateway

Configure PC2 with:

```text
192.168.20.254
```

instead of:

```text
192.168.20.1
```

Expected Result:

PC2 cannot communicate outside its network.

Reason:

Incorrect gateway configuration.

### Scenario 3: Trunk Failure

```bash
interface fa0/1
switchport mode access
```

Expected Result:

Inter-VLAN communication stops.

Reason:

The switch-router link is no longer carrying VLAN tags.

## Mini Challenge

Create VLAN 50 called MANAGEMENT.

Requirements:

- Create VLAN 50 on the switch
- Configure one PC in VLAN 50
- Create subinterface G0/0.50
- Configure IP address 192.168.50.1/24
- Configure the PC with a valid IP address
- Verify communication with VLAN 10

Suggested Configuration:

```bash
interface g0/0.50
encapsulation dot1Q 50
ip address 192.168.50.1 255.255.255.0
```

## Skills Learned

- Router-on-a-Stick
- Inter-VLAN Routing
- Router Subinterfaces
- 802.1Q Encapsulation
- Default Gateway Configuration
- Trunk Configuration
- Layer 3 Routing
- Troubleshooting Inter-VLAN Communication

## Conclusion

This lab introduced Router-on-a-Stick and demonstrated how a router can provide communication between multiple VLANs using a single physical interface. Inter-VLAN routing is a fundamental enterprise networking skill and forms the foundation for larger routed networks.

## Next Lab

Day 5 – VLAN Project

Topics:

- Multi-VLAN Enterprise Design
- Trunking
- Native VLAN
- Allowed VLANs
- Router-on-a-Stick
- End-to-End Troubleshooting

## Author

Muhammad Kausar

CCNA SRWE Hands-On Lab Series
