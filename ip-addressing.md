# IPv4 / VLAN Addressing Plan

**Campus block:** `10.50.0.0/16`

---

## Endpoint Requirements and Subnet Calculations

### Building A (Central Administration and Technology)

| Department | Devices | Count |
|---|---|---|
| Computer Science | Lab A (20 PCs) + 5 lecture PCs + 9 offices × (1 workstation + 1 printer) | 43 |
| Business | Lab B (10 PCs) + 5 lecture PCs + 9 offices × (1 workstation + 1 printer) | 33 |
| Administration | ~6 admin workstations + 3 printers | 9 |
| IT Sandbox | ~5 IT workstations + test equipment | 8 |

### Building B (Science and Engineering)

| Department | Devices | Count |
|---|---|---|
| Engineering | Lab C (15 PCs) + 5 lecture PCs + 7 offices × (1 workstation + 1 printer) | 34 |
| Mathematics & Science | 4 lecture PCs + 7 offices × (1 workstation + 1 printer) | 18 |

### Building C (Humanities and Communication)

| Department | Devices | Count |
|---|---|---|
| Humanities | 4 lecture PCs + 6 offices × (1 workstation + 1 printer) | 16 |
| Arts & Communication | 4 lecture PCs + 5 offices × (1 workstation + 1 printer) | 14 |

### Server Room (Building A)

| Device | Count |
|---|---|
| Application servers | 2 |
| Primary file server | 1 |
| Backup file server | 1 |
| Print server | 1 |
| NAS | 1 |
| KMS admin access device | 1 |
| DNS server | 1 |
| IDS/IPS | 1 |
| Total | 10 |

### DMZ

| Device | Count |
|---|---|
| Public web server | 1 |
| Proxy server | 1 |
| Total | 2 |

---

## VLSM Subnet Calculations

For each subnet: required hosts → add growth → determine host bits → CIDR prefix.

### Departmental VLANs (8 required)

| VLAN | Department | Hosts Needed | Growth | Total | Host Bits | Prefix | Usable |
|------|-----------|-------------|--------|-------|-----------|--------|--------|
| 10 | Computer Science | 43 | +19 | 62 | 6 | /26 | 62 |
| 20 | Business | 33 | +15 | 48 | 6 | /26 | 62 |
| 30 | Engineering | 34 | +15 | 49 | 6 | /26 | 62 |
| 40 | Math & Science | 18 | +12 | 30 | 5 | /27 | 30 |
| 50 | Humanities | 16 | +10 | 26 | 5 | /27 | 30 |
| 60 | Arts & Communication | 14 | +10 | 24 | 5 | /27 | 30 |
| 70 | Administration | 9 | +10 | 19 | 5 | /27 | 30 |
| 80 | IT Sandbox | 8 | +6 | 14 | 4 | /28 | 14 |

### Infrastructure VLANs

| VLAN | Purpose | Hosts Needed | Growth | Total | Host Bits | Prefix | Usable |
|------|---------|-------------|--------|-------|-----------|--------|--------|
| 90 | Servers | 10 | +10 | 20 | 5 | /27 | 30 |
| 100 | DMZ | 2 | +4 | 6 | 3 | /29 | 6 |
| 111 | Wireless, Bldg A | ~30 | +32 | 62 | 6 | /26 | 62 |
| 112 | Wireless, Bldg B | ~20 | +30 | 50 | 6 | /26 | 62 |
| 113 | Wireless, Bldg C + Outdoor | ~25 | +30 | 55 | 6 | /26 | 62 |
| 120 | Management, Bldg A | ~8 | +6 | 14 | 4 | /28 | 14 |
| 121 | Management, Bldg B | ~5 | +5 | 10 | 4 | /28 | 14 |
| 122 | Management, Bldg C | ~5 | +5 | 10 | 4 | /28 | 14 |

### Point-to-Point Links

| Link | Hosts | Prefix |
|------|-------|--------|
| CORE-SW1 to BLDGB-SW1 | 2 | /30 |
| CORE-SW1 to BLDGC-SW1 | 2 | /30 |
| CORE-SW1 to CAMPUS-FW1 (inside) | 2 | /30 |
| CAMPUS-FW1 (outside) to EDGE-RTR1 | 2 | /30 |
| EDGE-RTR1 to ISP-RTR1 | 2 | /30 |

---

## Complete Addressing Table

### Departmental VLANs

| VLAN | Department / Purpose | Network Address | CIDR | Subnet Mask | Default Gateway | Usable Host Range | Broadcast |
|------|---------------------|----------------|------|-------------|-----------------|-------------------|-----------|
| 10 | Computer Science | 10.50.10.0 | /26 | 255.255.255.192 | 10.50.10.1 | 10.50.10.2 - 10.50.10.62 | 10.50.10.63 |
| 20 | Business | 10.50.20.0 | /26 | 255.255.255.192 | 10.50.20.1 | 10.50.20.2 - 10.50.20.62 | 10.50.20.63 |
| 30 | Engineering | 10.50.30.0 | /26 | 255.255.255.192 | 10.50.30.1 | 10.50.30.2 - 10.50.30.62 | 10.50.30.63 |
| 40 | Math & Science | 10.50.40.0 | /27 | 255.255.255.224 | 10.50.40.1 | 10.50.40.2 - 10.50.40.30 | 10.50.40.31 |
| 50 | Humanities | 10.50.50.0 | /27 | 255.255.255.224 | 10.50.50.1 | 10.50.50.2 - 10.50.50.30 | 10.50.50.31 |
| 60 | Arts & Communication | 10.50.60.0 | /27 | 255.255.255.224 | 10.50.60.1 | 10.50.60.2 - 10.50.60.30 | 10.50.60.31 |
| 70 | Administration | 10.50.70.0 | /27 | 255.255.255.224 | 10.50.70.1 | 10.50.70.2 - 10.50.70.30 | 10.50.70.31 |
| 80 | IT Sandbox | 10.50.80.0 | /28 | 255.255.255.240 | 10.50.80.1 | 10.50.80.2 - 10.50.80.14 | 10.50.80.15 |

### Infrastructure VLANs

| VLAN | Purpose | Network Address | CIDR | Subnet Mask | Default Gateway | Usable Host Range | Broadcast |
|------|---------|----------------|------|-------------|-----------------|-------------------|-----------|
| 90 | Servers | 10.50.90.0 | /27 | 255.255.255.224 | 10.50.90.1 | 10.50.90.2 - 10.50.90.30 | 10.50.90.31 |
| 100 | DMZ | 10.50.100.0 | /29 | 255.255.255.248 | 10.50.100.1 | 10.50.100.2 - 10.50.100.6 | 10.50.100.7 |
| 111 | Wireless, Bldg A | 10.50.111.0 | /26 | 255.255.255.192 | 10.50.111.1 | 10.50.111.2 - 10.50.111.62 | 10.50.111.63 |
| 112 | Wireless, Bldg B | 10.50.112.0 | /26 | 255.255.255.192 | 10.50.112.1 | 10.50.112.2 - 10.50.112.62 | 10.50.112.63 |
| 113 | Wireless, Bldg C / Outdoor | 10.50.113.0 | /26 | 255.255.255.192 | 10.50.113.1 | 10.50.113.2 - 10.50.113.62 | 10.50.113.63 |
| 120 | Net Management, Bldg A | 10.50.120.0 | /28 | 255.255.255.240 | 10.50.120.1 | 10.50.120.2 - 10.50.120.14 | 10.50.120.15 |
| 121 | Net Management, Bldg B | 10.50.120.16 | /28 | 255.255.255.240 | 10.50.120.17 | 10.50.120.18 - 10.50.120.30 | 10.50.120.31 |
| 122 | Net Management, Bldg C | 10.50.120.32 | /28 | 255.255.255.240 | 10.50.120.33 | 10.50.120.34 - 10.50.120.46 | 10.50.120.47 |

### Point-to-Point / Backbone Links

| Link | Network | CIDR | Subnet Mask | Endpoint A (IP) | Endpoint B (IP) |
|------|---------|------|-------------|-----------------|-----------------|
| CORE-SW1 to BLDGB-SW1 | 10.50.200.0 | /30 | 255.255.255.252 | CORE-SW1 Gig0/1 = 10.50.200.1 | BLDGB-SW1 Gig0/1 = 10.50.200.2 |
| CORE-SW1 to BLDGC-SW1 | 10.50.200.4 | /30 | 255.255.255.252 | CORE-SW1 Gig0/2 = 10.50.200.5 | BLDGC-SW1 Gig0/1 = 10.50.200.6 |
| CORE-SW1 to CAMPUS-FW1 | 10.50.200.8 | /30 | 255.255.255.252 | CORE-SW1 Fa0/24 = 10.50.200.9 | CAMPUS-FW1 G0/0 = 10.50.200.10 |
| CAMPUS-FW1 to EDGE-RTR1 | 10.50.200.12 | /30 | 255.255.255.252 | CAMPUS-FW1 G0/1 = 10.50.200.13 | EDGE-RTR1 G0/0 = 10.50.200.14 |
| EDGE-RTR1 to ISP-RTR1 | 203.0.113.0 | /30 | 255.255.255.252 | EDGE-RTR1 G0/1 = 203.0.113.1 | ISP-RTR1 G0/0 = 203.0.113.2 |

---

## Server Room Static Assignments (VLAN 90, 10.50.90.0/27)

| Hostname | IP Address | Role |
|----------|-----------|------|
| APP-SRV1 | 10.50.90.2 | Application Server 1 |
| APP-SRV2 | 10.50.90.3 | Application Server 2 |
| FILE-SRV1 | 10.50.90.4 | Primary File Server |
| BACKUP-SRV1 | 10.50.90.5 | Backup File Server |
| PRINT-SRV1 | 10.50.90.6 | Print Server |
| NAS-01 | 10.50.90.7 | Network Attached Storage |
| KMS-01 | 10.50.90.8 | KMS Administrative Access |
| DNS-SRV1 | 10.50.90.9 | DNS Server |
| IDS-IPS1 | 10.50.90.10 | Hardware IDS/IPS |

Gateway: 10.50.90.1 (CORE-SW1 VLAN 90 SVI) | Mask: 255.255.255.224 | DNS: 10.50.90.9

## DMZ Static Assignments (VLAN 100, 10.50.100.0/29)

| Hostname | IP Address | Role |
|----------|-----------|------|
| WEB-DMZ1 | 10.50.100.2 | Public-facing HTML/Web Server |
| PROXY-DMZ1 | 10.50.100.3 | Proxy Server |

Gateway: 10.50.100.1 (CAMPUS-FW1 G0/2) | Mask: 255.255.255.248

---

## DHCP Pools (configured on each building's L3 switch)

| Pool Name | Network | Excluded (static/gw) | Gateway | DNS |
|-----------|---------|----------------------|---------|-----|
| POOL-COMPSCI | 10.50.10.0/26 | 10.50.10.1 | 10.50.10.1 | 10.50.90.9 |
| POOL-BUSINESS | 10.50.20.0/26 | 10.50.20.1 | 10.50.20.1 | 10.50.90.9 |
| POOL-ENGINEERING | 10.50.30.0/26 | 10.50.30.1 | 10.50.30.1 | 10.50.90.9 |
| POOL-MATHSCI | 10.50.40.0/27 | 10.50.40.1 | 10.50.40.1 | 10.50.90.9 |
| POOL-HUMANITIES | 10.50.50.0/27 | 10.50.50.1 | 10.50.50.1 | 10.50.90.9 |
| POOL-ARTSCOMM | 10.50.60.0/27 | 10.50.60.1 | 10.50.60.1 | 10.50.90.9 |
| POOL-ADMIN | 10.50.70.0/27 | 10.50.70.1 | 10.50.70.1 | 10.50.90.9 |
| POOL-ITSANDBOX | 10.50.80.0/28 | 10.50.80.1 | 10.50.80.1 | 10.50.90.9 |
| POOL-WIFI-A | 10.50.111.0/26 | 10.50.111.1 | 10.50.111.1 | 10.50.90.9 |
| POOL-WIFI-B | 10.50.112.0/26 | 10.50.112.1 | 10.50.112.1 | 10.50.90.9 |
| POOL-WIFI-C | 10.50.113.0/26 | 10.50.113.1 | 10.50.113.1 | 10.50.90.9 |

---

## Verification — No Subnet Overlap

All subnets use distinct third-octet values (10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 111, 112, 113, 200) except the three management subnets which are contiguous /28 blocks inside 10.50.120.0/24:

- 10.50.120.0/28  = .0 - .15
- 10.50.120.16/28 = .16 - .31
- 10.50.120.32/28 = .32 - .47

No overlap exists between any two subnets.
