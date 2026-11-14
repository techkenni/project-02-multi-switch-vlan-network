# Project 02: Multi-Switch VLAN Network
## Technical implementation and verification report

**Portfolio:** KenniTech Networking Project Portfolio  
**Implementation date:** 14 November 2026  
**Platform:** Cisco Packet Tracer on macOS

## 1. Executive summary

This project extends a departmental VLAN design across two Layer 2 switches. Each department has one PC connected to each switch, allowing devices in the same VLAN to communicate across the inter-switch trunk. A Cisco 2911 router connects to SW1 and provides inter-VLAN routing through 802.1Q subinterfaces on GigabitEthernet0/0.

The router also provides DHCP for the Sales, IT and HR subnets. Verification evidence shows six automatic DHCP bindings, successful pings to local gateways and remote departmental endpoints, and a completed Spanning Tree Protocol (STP) root bridge election. SW1 is the root bridge for VLANs 10, 20 and 30. SW2 uses GigabitEthernet0/1 as its Root Port for all three VLANs.

The router and switch configurations were saved to startup configuration. The CLI evidence shows the save operation completing with `[OK]`.

## 2. Objectives and scope

- Create three departmental VLANs: Sales, IT and HR.
- Connect six PCs across two Cisco 2960 switches, with one PC per department on each switch.
- Carry VLANs 10, 20 and 30 over 802.1Q trunk links.
- Configure router-on-a-stick inter-VLAN routing on a Cisco 2911 router.
- Provide automatic IPv4 addressing through three DHCP pools.
- Verify connectivity within and between departmental VLANs.
- Configure and verify SW1 as the STP root bridge for all three VLANs.
- Save router and switch configurations to startup configuration.

## 3. Topology and equipment

The topology contains:
- R1-KenniTech, a Cisco 2911 router.
- SW1-KenniTech, a Cisco 2960-24TT switch.
- SW2-KenniTech, a Cisco 2960-24TT switch.
- Six PCs, two each for Sales, IT and HR.

R1 connects to SW1. SW1 connects to SW2 through an inter-switch trunk. Each switch has three departmental access ports.

### 3.1 Link and port plan

| From | Interface | To | Interface | Link role |
|---|---|---|---|---|
| R1 | GigabitEthernet0/0 | SW1 | GigabitEthernet0/1 | 802.1Q router uplink |
| SW1 | GigabitEthernet0/2 | SW2 | GigabitEthernet0/1 | 802.1Q inter-switch trunk |
| SW1 | FastEthernet0/1 | Sales PC1 | FastEthernet0 | Access, VLAN 10 |
| SW1 | FastEthernet0/2 | IT PC1 | FastEthernet0 | Access, VLAN 20 |
| SW1 | FastEthernet0/3 | HR PC1 | FastEthernet0 | Access, VLAN 30 |
| SW2 | FastEthernet0/1 | Sales PC2 | FastEthernet0 | Access, VLAN 10 |
| SW2 | FastEthernet0/2 | IT PC2 | FastEthernet0 | Access, VLAN 20 |
| SW2 | FastEthernet0/3 | HR PC2 | FastEthernet0 | Access, VLAN 30 |

## 4. VLAN and IPv4 addressing plan

| Department | VLAN ID | Network / mask | Default gateway | DHCP clients observed |
|---|---:|---|---|---|
| Sales | 10 | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.11, 192.168.10.12 |
| IT | 20 | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.11, 192.168.20.12 |
| HR | 30 | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.11, 192.168.30.12 |

The router DHCP configuration excludes addresses .1 through .10 in each subnet. The .1 address in each subnet is used as its default gateway.

## 5. Switch configuration

### 5.1 VLAN creation and access ports

VLAN 10 is named SALES, VLAN 20 is named IT, and VLAN 30 is named HR. On both switches, FastEthernet0/1 is assigned to VLAN 10, FastEthernet0/2 to VLAN 20 and FastEthernet0/3 to VLAN 30.

```cisco
vlan 10
 name SALES
exit
vlan 20
 name IT
exit
vlan 30
 name HR
exit

interface fastEthernet 0/1
 description SALES-PC
 switchport mode access
 switchport access vlan 10
exit
interface fastEthernet 0/2
 description IT-PC
 switchport mode access
 switchport access vlan 20
exit
interface fastEthernet 0/3
 description HR-PC
 switchport mode access
 switchport access vlan 30
exit
```

The switch VLAN brief output confirms the three departmental VLANs are active and the expected FastEthernet ports are assigned.

### 5.2 802.1Q trunk configuration

The router uplink on SW1 GigabitEthernet0/1 and the inter-switch link between SW1 GigabitEthernet0/2 and SW2 GigabitEthernet0/1 are configured as trunks. VLANs 10, 20 and 30 are explicitly allowed.

SW1:

```cisco
interface gigabitEthernet 0/1
 description TRUNK-TO-R1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface gigabitEthernet 0/2
 description TRUNK-TO-SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

SW2:

```cisco
interface gigabitEthernet 0/1
 description TRUNK-TO-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

The trunk verification output reports 802.1Q encapsulation, trunking status and VLANs 10, 20 and 30 as allowed and active.

## 6. Router-on-a-stick configuration

The router's physical GigabitEthernet0/0 interface is enabled. Three subinterfaces are created, one for each VLAN. Each subinterface uses IEEE 802.1Q encapsulation and the first usable address in its departmental subnet as the default gateway.

```cisco
interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
```

The router's `show ip interface brief` output shows GigabitEthernet0/0 and subinterfaces GigabitEthernet0/0.10, .20 and .30 as up/up. The subinterfaces provide the Layer 3 gateways required for traffic to move between VLANs.

## 7. DHCP configuration and verification

Three DHCP pools are configured on R1, one for each department. Each pool advertises its departmental /24 network, the correct default gateway and DNS server 8.8.8.8. Addresses .1 through .10 are excluded from dynamic assignment in each subnet.

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10

ip dhcp pool SALES
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

ip dhcp pool IT
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit

ip dhcp pool HR
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
exit
```

The DHCP binding output confirms six automatic leases:

| Department | PC | DHCP address | Lease type |
|---|---|---|---|
| Sales | Sales PC1 | 192.168.10.11 | Automatic |
| Sales | Sales PC2 | 192.168.10.12 | Automatic |
| IT | IT PC1 | 192.168.20.11 | Automatic |
| IT | IT PC2 | 192.168.20.12 | Automatic |
| HR | HR PC1 | 192.168.30.11 | Automatic |
| HR | HR PC2 | 192.168.30.12 | Automatic |

## 8. Connectivity testing

Endpoint tests were carried out from Packet Tracer PC command prompts. The supplied evidence shows successful communication with local gateways and remote departmental hosts.

| Source | Destination | Purpose | Observed result |
|---|---|---|---|
| Sales PC1 (192.168.10.11) | Sales PC2 (192.168.10.12) | Same-VLAN communication | 4 replies, 0% loss |
| Sales PC1 | Sales gateway (192.168.10.1) | Gateway reachability | 4 replies, 0% loss |
| Sales PC1 | IT PC1 (192.168.20.11) | Inter-VLAN routing | 4 replies, 0% loss |
| Sales PC1 | HR PC1 (192.168.30.11) | Inter-VLAN routing | 4 replies, 0% loss |
| IT PC1 (192.168.20.11) | IT PC2 (192.168.20.12) | Same-VLAN communication | 4 replies, 0% loss |
| IT PC1 | IT gateway (192.168.20.1) | Gateway reachability | 4 replies, 0% loss |
| IT PC1 | HR PC1 (192.168.30.11) | Inter-VLAN routing | Final correctly addressed test: 4 replies, 0% loss |

### 8.1 Address correction during testing

An earlier test used 198.168.30.11, which is not part of the configured HR subnet. The HR network is 192.168.30.0/24. The corrected test to 192.168.30.11 returned four replies with zero packet loss. The unsuccessful attempt to the mistyped address was not evidence of a routing failure.

## 9. Spanning Tree Protocol

STP was inspected on both switches for VLANs 10, 20 and 30. Initially, SW2 was the root bridge for the VLANs. SW1 was then configured as the preferred root bridge using the following commands:

```cisco
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
spanning-tree vlan 30 root primary
```

Final SW1 verification showed that the switch reports “This bridge is the root” for each VLAN. The root bridge MAC address is 0060.3E97.4B03.

Final SW2 verification showed:
- Root ID address 0060.3E97.4B03 for VLANs 10, 20 and 30.
- GigabitEthernet0/1 selected as the Root Port for each VLAN.
- Root Port state Forwarding (FWD), with path cost 4.
- Departmental access ports shown as Designated Forwarding ports.

| VLAN | Department | Root bridge | SW2 Root Port | Port state |
|---:|---|---|---|---|
| 10 | Sales | SW1 | GigabitEthernet0/1 | Root FWD |
| 20 | IT | SW1 | GigabitEthernet0/1 | Root FWD |
| 30 | HR | SW1 | GigabitEthernet0/1 | Root FWD |

## 10. Troubleshooting and learning points

### 10.1 Trunk port inconsistency warning

During switch-to-switch trunk setup, SW2 displayed a spanning-tree PVID/port-type inconsistency message on GigabitEthernet0/1. The interface was explicitly configured as a trunk and VLANs 10, 20 and 30 were allowed. The subsequent trunk verification output showed the link trunking and the three departmental VLANs active. This demonstrates why both ends of an inter-switch link must have compatible trunk configurations.

### 10.2 STP root bridge election

The initial STP output showed SW2 as the root bridge. Configuring SW1 with the spanning-tree VLAN root primary commands changed the election. The final outputs on SW1 and SW2 confirm the new root bridge and the Root Port selection on SW2.

### 10.3 IP address accuracy

The failed ping to 198.168.30.11 was caused by an incorrect destination address. The HR network is 192.168.30.0/24. Repeating the test with 192.168.30.11 succeeded. This highlights the value of checking addresses before changing device configuration.

## 11. Configuration backup and persistence

The router and switch configurations were saved using:

```cisco
copy running-config startup-config
```

The CLI screenshots show the save process completing with `[OK]`.

Recommended persistence check: save the Packet Tracer file, close and reopen it, then repeat the key verification commands. Confirm VLAN membership, trunk status, router subinterface status, DHCP bindings and STP root bridge information after reopening.

## 12. Conclusion

Project 02 implements a multi-switch departmental network with three VLANs, 802.1Q trunking, router-on-a-stick inter-VLAN routing, DHCP and STP root bridge control. Six DHCP leases were recorded, endpoint tests demonstrate successful same-VLAN and inter-VLAN communication, and the final STP evidence confirms SW1 as the root bridge for VLANs 10, 20 and 30. SW2 uses GigabitEthernet0/1 as its Root Port for each VLAN. The router and switch configurations were saved to startup configuration.

## Appendix A. Useful verification commands

```cisco
! Switch verification
show vlan brief
show interfaces trunk
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
show running-config

! Router verification
show ip interface brief
show ip dhcp pool
show ip dhcp binding
show running-config

! Save configuration
copy running-config startup-config
```
