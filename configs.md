# Key Configuration Excerpts

Selected configurations from the campus network devices. These are the core settings that make inter-VLAN routing, OSPF, DHCP, NAT, and security work across the three buildings.

## CORE-SW1 (Building A, Layer 3 Core Switch)

### VLAN Definitions

```
vlan 10
 name CompSci
vlan 20
 name Business
vlan 30
 name Engineering
vlan 40
 name MathScience
vlan 50
 name Humanities
vlan 60
 name ArtsComm
vlan 70
 name Administration
vlan 80
 name IT-Sandbox
vlan 90
 name Servers
vlan 111
 name Wireless-BldgA
vlan 112
 name Wireless-BldgB
vlan 113
 name Wireless-BldgC
vlan 120
 name Mgmt-BldgA
vlan 121
 name Mgmt-BldgB
vlan 122
 name Mgmt-BldgC
```

### SVI Interfaces (Inter-VLAN Routing)

```
interface vlan 10
 ip address 10.50.10.1 255.255.255.192
 no shutdown
interface vlan 20
 ip address 10.50.20.1 255.255.255.192
 no shutdown
interface vlan 30
 ip address 10.50.30.1 255.255.255.192
 no shutdown
interface vlan 40
 ip address 10.50.40.1 255.255.255.224
 no shutdown
interface vlan 50
 ip address 10.50.50.1 255.255.255.224
 no shutdown
interface vlan 60
 ip address 10.50.60.1 255.255.255.224
 no shutdown
interface vlan 70
 ip address 10.50.70.1 255.255.255.224
 no shutdown
interface vlan 80
 ip address 10.50.80.1 255.255.255.240
 no shutdown
interface vlan 90
 ip address 10.50.90.1 255.255.255.224
 no shutdown
interface vlan 111
 ip address 10.50.111.1 255.255.255.192
 no shutdown
```

### Backbone Links

```
interface GigabitEthernet0/1
 description Link to BLDGB-SW1
 no switchport
 ip address 10.50.200.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 description Link to BLDGC-SW1
 no switchport
 ip address 10.50.200.5 255.255.255.252
 no shutdown

interface FastEthernet0/24
 description Link to CAMPUS-FW1 inside
 no switchport
 ip address 10.50.200.9 255.255.255.252
 no shutdown
```

### OSPF

```
router ospf 1
 router-id 1.1.1.1
 network 10.50.10.0 0.0.0.63 area 0
 network 10.50.20.0 0.0.0.63 area 0
 network 10.50.30.0 0.0.0.63 area 0
 network 10.50.40.0 0.0.0.31 area 0
 network 10.50.50.0 0.0.0.31 area 0
 network 10.50.60.0 0.0.0.31 area 0
 network 10.50.70.0 0.0.0.31 area 0
 network 10.50.80.0 0.0.0.15 area 0
 network 10.50.90.0 0.0.0.31 area 0
 network 10.50.111.0 0.0.0.63 area 0
 network 10.50.200.0 0.0.0.3 area 0
 network 10.50.200.4 0.0.0.3 area 0
 network 10.50.200.8 0.0.0.3 area 0
 default-information originate
```

### DHCP Pools (selected)

```
ip dhcp excluded-address 10.50.10.1
ip dhcp excluded-address 10.50.20.1
ip dhcp excluded-address 10.50.30.1
ip dhcp excluded-address 10.50.40.1
ip dhcp excluded-address 10.50.90.1 10.50.90.15

ip dhcp pool POOL-COMPSCI
 network 10.50.10.0 255.255.255.192
 default-router 10.50.10.1
 dns-server 10.50.90.9

ip dhcp pool POOL-BUSINESS
 network 10.50.20.0 255.255.255.192
 default-router 10.50.20.1
 dns-server 10.50.90.9

ip dhcp pool POOL-ENGINEERING
 network 10.50.30.0 255.255.255.192
 default-router 10.50.30.1
 dns-server 10.50.90.9
```

## BLDGB-SW1 (Building B, Layer 3 Switch)

```
interface GigabitEthernet0/1
 description Link to CORE-SW1
 no switchport
 ip address 10.50.200.2 255.255.255.252
 no shutdown

router ospf 1
 router-id 2.2.2.2
 network 10.50.200.0 0.0.0.3 area 0
 network 10.50.112.0 0.0.0.63 area 0
 network 10.50.120.16 0.0.0.15 area 0
```

## BLDGC-SW1 (Building C, Layer 3 Switch)

```
interface GigabitEthernet0/1
 description Link to CORE-SW1
 no switchport
 ip address 10.50.200.6 255.255.255.252
 no shutdown

router ospf 1
 router-id 3.3.3.3
 network 10.50.200.4 0.0.0.3 area 0
 network 10.50.113.0 0.0.0.63 area 0
 network 10.50.120.32 0.0.0.15 area 0
```

## Access Switch Trunk Configuration (example: BLDGA-F1-SW1)

```
interface GigabitEthernet0/1
 description Uplink to CORE-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,70,80,90,111,120

interface FastEthernet0/1
 description PC-A-CS01
 switchport mode access
 switchport access vlan 10

interface FastEthernet0/2
 description PC-A-BUS01
 switchport mode access
 switchport access vlan 20
```

## EDGE-RTR1 (Internet Edge Router)

```
interface GigabitEthernet0/0
 description Link to CAMPUS-FW1 outside
 ip address 10.50.200.14 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 description Link to ISP
 ip address 203.0.113.1 255.255.255.252
 no shutdown

ip nat inside source list 1 interface GigabitEthernet0/1 overload

access-list 1 permit 10.50.0.0 0.0.255.255

interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside

ip route 0.0.0.0 0.0.0.0 203.0.113.2

router ospf 1
 router-id 4.4.4.4
 network 10.50.200.12 0.0.0.3 area 0
 default-information originate
```

## CAMPUS-FW1 (Firewall)

```
interface GigabitEthernet0/0
 description Inside (to CORE-SW1)
 ip address 10.50.200.10 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 description Outside (to EDGE-RTR1)
 ip address 10.50.200.13 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 description DMZ
 ip address 10.50.100.1 255.255.255.248
 no shutdown
```

## ACL Examples

### Block external hosts from reaching internal VLANs

```
access-list 100 permit tcp any host 10.50.100.2 eq 80
access-list 100 deny ip any 10.50.0.0 0.0.255.255
access-list 100 permit ip any any
```

### IT Sandbox isolation from Administration

```
access-list 110 deny ip 10.50.80.0 0.0.0.15 10.50.70.0 0.0.0.31
access-list 110 permit ip any any
```
