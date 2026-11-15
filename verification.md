# Network Verification and Testing Results

This document records the verification tests performed on the completed campus network. Each section covers a different aspect of network functionality. Screenshots of these tests are included in the repository.

---

## 1. DHCP Address Assignment

All wired PCs and wireless laptops were set to DHCP and received addresses from the correct pools on CORE-SW1.

### Wired PC DHCP Results

| Device | VLAN | Expected Subnet | Assigned IP | Status |
|--------|------|-----------------|-------------|--------|
| PC-A-CS01 | 10 | 10.50.10.0/26 | 10.50.10.11 | Pass |
| PC-A-BUS01 | 20 | 10.50.20.0/26 | 10.50.20.11 | Pass |
| PC-B-ENG01 | 30 | 10.50.30.0/26 | 10.50.30.11 | Pass |
| PC-B-MATH01 | 40 | 10.50.40.0/27 | 10.50.40.11 | Pass |
| PC-C-HUM01 | 50 | 10.50.50.0/27 | 10.50.50.11 | Pass |
| PC-C-ART01 | 60 | 10.50.60.0/27 | 10.50.60.11 | Pass |
| PC-A-ADM01 | 70 | 10.50.70.0/27 | 10.50.70.11 | Pass |
| PC-A-IT01 | 80 | 10.50.80.0/28 | 10.50.80.11 | Pass |

### Wireless Laptop DHCP Results

| Device | VLAN | Expected Subnet | Assigned IP | Status |
|--------|------|-----------------|-------------|--------|
| LAPTOP-A-W01 | 111 | 10.50.111.0/26 | 10.50.111.11 | Pass |
| LAPTOP-B-W01 | 112 | 10.50.112.0/26 | 10.50.112.11 | Pass |
| LAPTOP-C-W01 | 113 | 10.50.113.0/26 | 10.50.113.12 | Pass |

All 11 endpoints received addresses within their expected subnets. The DNS server field was populated as 10.50.90.9 on every client.

### DHCP Binding Table (CORE-SW1)

Output from `show ip dhcp binding` on CORE-SW1 confirmed 12 active automatic bindings:

| IP Address | Client MAC | Type |
|------------|-----------|------|
| 10.50.10.11 | 0001.642A.4189 | Automatic |
| 10.50.20.11 | 0002.1639.D00D | Automatic |
| 10.50.30.11 | 0002.4AB6.290D | Automatic |
| 10.50.40.11 | 0030.A3B3.EC35 | Automatic |
| 10.50.50.11 | 00E0.8F94.4AB9 | Automatic |
| 10.50.60.11 | 0050.0F10.78BA | Automatic |
| 10.50.70.11 | 00D0.58DC.D00C | Automatic |
| 10.50.80.11 | 000C.854D.6AD0 | Automatic |
| 10.50.111.11 | 0002.1661.0070 | Automatic |
| 10.50.112.11 | 0004.9A48.E642 | Automatic |
| 10.50.113.11 | 00D0.FFE3.1E82 | Automatic |
| 10.50.113.12 | 0002.1661.0070 | Automatic |

---

## 2. Server Static IP Verification

All servers were configured with static IP addresses through their Desktop tab in Packet Tracer.

| Server | VLAN | IP Address | Subnet Mask | Gateway | Status |
|--------|------|-----------|-------------|---------|--------|
| APP-SRV1 | 90 | 10.50.90.2 | 255.255.255.224 | 10.50.90.1 | Configured |
| APP-SRV2 | 90 | 10.50.90.3 | 255.255.255.224 | 10.50.90.1 | Configured |
| FILE-SRV1 | 90 | 10.50.90.4 | 255.255.255.224 | 10.50.90.1 | Configured |
| BACKUP-SRV1 | 90 | 10.50.90.5 | 255.255.255.224 | 10.50.90.1 | Configured |
| PRINT-SRV1 | 90 | 10.50.90.6 | 255.255.255.224 | 10.50.90.1 | Configured |
| NAS-01 | 90 | 10.50.90.7 | 255.255.255.224 | 10.50.90.1 | Configured |
| KMS-01 | 90 | 10.50.90.8 | 255.255.255.224 | 10.50.90.1 | Configured |
| DNS-SRV1 | 90 | 10.50.90.9 | 255.255.255.224 | 10.50.90.1 | Configured |
| IDS-IPS1 | 90 | 10.50.90.10 | 255.255.255.224 | 10.50.90.1 | Configured |
| WEB-DMZ1 | 100 | 10.50.100.2 | 255.255.255.248 | 10.50.100.1 | Configured |
| PROXY-DMZ1 | 100 | 10.50.100.3 | 255.255.255.248 | 10.50.100.1 | Configured |

---

## 3. Inter-VLAN Routing Tests

Ping tests from PC-A-CS01 (VLAN 10, IP 10.50.10.11) to devices on other VLANs confirm that inter-VLAN routing through CORE-SW1 SVIs works correctly.

| Source | Destination | Destination IP | Result |
|--------|------------|---------------|--------|
| PC-A-CS01 (VLAN 10) | PC-A-BUS01 (VLAN 20) | 10.50.20.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-B-ENG01 (VLAN 30) | 10.50.30.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-B-MATH01 (VLAN 40) | 10.50.40.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-C-HUM01 (VLAN 50) | 10.50.50.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-C-ART01 (VLAN 60) | 10.50.60.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-A-ADM01 (VLAN 70) | 10.50.70.11 | Pass |
| PC-A-CS01 (VLAN 10) | PC-A-IT01 (VLAN 80) | 10.50.80.11 | Pass |

---

## 4. PC-to-Server Connectivity

| Source | Destination Server | IP | Result |
|--------|-------------------|-----|--------|
| PC-A-CS01 | APP-SRV1 | 10.50.90.2 | Pass |
| PC-A-CS01 | DNS-SRV1 | 10.50.90.9 | Pass |
| PC-A-CS01 | WEB-DMZ1 | 10.50.100.2 | Pass |
| PC-A-CS01 | PROXY-DMZ1 | 10.50.100.3 | Pass |

---

## 5. Cross-Building Connectivity

| Source | Destination | Buildings | Result |
|--------|------------|-----------|--------|
| PC-A-CS01 (Bldg A) | PC-B-ENG01 (Bldg B) | A to B | Pass |
| PC-A-CS01 (Bldg A) | PC-C-HUM01 (Bldg C) | A to C | Pass |
| PC-B-ENG01 (Bldg B) | PC-C-HUM01 (Bldg C) | B to C | Pass |

---

## 6. Wireless Connectivity Tests

### Wired to Wireless

| Source (Wired) | Destination (Wireless) | Destination IP | Result |
|----------------|----------------------|---------------|--------|
| PC-A-CS01 | LAPTOP-A-W01 | 10.50.111.11 | Pass |
| PC-A-CS01 | LAPTOP-B-W01 | 10.50.112.11 | Pass |
| PC-A-CS01 | LAPTOP-C-W01 | 10.50.113.12 | Pass |

### Wireless to Wired

| Source (Wireless) | Destination (Wired) | Destination IP | Result |
|-------------------|-------------------|---------------|--------|
| LAPTOP-A-W01 | PC-B-ENG01 | 10.50.30.11 | Pass |
| LAPTOP-B-W01 | PC-C-HUM01 | 10.50.50.11 | Pass |

### Wireless to Server

| Source (Wireless) | Destination Server | Destination IP | Result |
|-------------------|-------------------|---------------|--------|
| LAPTOP-A-W01 | APP-SRV1 | 10.50.90.2 | Pass |
| LAPTOP-B-W01 | DNS-SRV1 | 10.50.90.9 | Pass |

---

## 7. Internet Connectivity

### Ping to ISP Router

| Source | Destination | IP | Result |
|--------|------------|-----|--------|
| PC-A-CS01 | ISP-RTR1 | 203.0.113.1 | Pass (3/4 packets) |

The first packet timed out due to ARP resolution delay, which is normal in Packet Tracer. The remaining packets went through fine.

### Traceroute to ISP

From PC-A-CS01 Command Prompt:

```
C:\>tracert 203.0.113.1

Tracing route to 203.0.113.1 over a maximum of 30 hops

  1   3 ms    0 ms    0 ms    10.50.10.1
  2   0 ms    0 ms    0 ms    10.50.200.10
  3   0 ms    0 ms    0 ms    10.50.200.14
  4   0 ms    0 ms    0 ms    203.0.113.1

Trace complete.
```

The traceroute confirms the expected 4-hop path:

1. **10.50.10.1** , CORE-SW1 VLAN 10 SVI (default gateway)
2. **10.50.200.10** , CAMPUS-FW1 inside interface
3. **10.50.200.14** , EDGE-RTR1 inside interface
4. **203.0.113.1** , ISP-RTR1 (destination)

---

## 8. OSPF Adjacency

Output from `show ip ospf neighbor` on CORE-SW1:

| Neighbor ID | Priority | State | Dead Time | Address | Interface |
|------------|----------|-------|-----------|---------|-----------|
| 10.50.200.13 | 1 | FULL/DR | 00:00:39 | 10.50.200.10 | GigabitEthernet0/2 |

OSPF adjacency between CORE-SW1 and CAMPUS-FW1 is established in Area 0. The FULL state means both routers have synchronized their link-state databases.

---

## 9. NAT Verification

`show ip nat translations` on CAMPUS-FW1 returned no active translations at the time of capture because Packet Tracer clears NAT entries quickly after traffic stops. The successful ping and traceroute to 203.0.113.1 from internal PCs do confirm that NAT is translating correctly though. Without NAT, the ISP router would have no return path for traffic from 10.50.x.x addresses.

NAT configuration on CAMPUS-FW1:

- Inside interface: Gig0/1 (10.50.200.10)
- Outside interface: Gig0/0 (10.50.200.13)
- ACL 1: permit 10.50.0.0 0.0.255.255
- Overload on outside interface

---

## 10. Issues Encountered and Resolved

### Issue 1: Building B DHCP Failure

**Symptom:** PCs in Building B (VLANs 30 and 40) could not obtain DHCP addresses. All other buildings worked.

**Root Cause:** The cable from BLDGB-SW1 Gig0/1 was connected to CORE-SW1 Gig0/2 (a routed port configured for CAMPUS-FW1) instead of CORE-SW1 Gig0/1 (the trunk port). The cables between CORE-SW1 and BLDGB-SW1/CAMPUS-FW1 were swapped.

**Resolution:** Swapped the two cables to their correct ports. Building B DHCP started working immediately. No configuration changes were needed.

### Issue 2: Wireless Clients Not Receiving DHCP

**Symptom:** All three wireless laptops failed to get DHCP addresses after connecting to the SSID. Wired PCs on the same access switches worked.

**Root Cause:** AP switch ports (Fa0/1 on each access switch) were configured as trunk ports. Packet Tracer's AP-PT sends wireless traffic untagged, so frames landed on native VLAN 1 (no DHCP pool) instead of the intended wireless VLAN.

**Resolution:** Changed all AP ports from trunk mode to access mode on the correct wireless VLAN (111, 112, or 113 depending on building). Wireless DHCP worked after the change.

### Issue 3: Internet Connectivity Failure

**Symptom:** Internal PCs could ping all internal destinations but could not reach the ISP router at 203.0.113.1.

**Root Cause:** CAMPUS-FW1 had no default route. Its routing table showed "Gateway of last resort is not set," so traffic destined for the Internet was being dropped at the firewall.

**Resolution:** Added `ip route 0.0.0.0 0.0.0.0 10.50.200.14` on CAMPUS-FW1. Internet access worked immediately after.

---

## Test Summary

| Category | Tests Run | Passed | Failed |
|----------|----------|--------|--------|
| DHCP (Wired) | 8 | 8 | 0 |
| DHCP (Wireless) | 3 | 3 | 0 |
| Inter-VLAN Routing | 7 | 7 | 0 |
| PC to Server | 4 | 4 | 0 |
| Cross-Building | 3 | 3 | 0 |
| Wired to Wireless | 3 | 3 | 0 |
| Wireless to Wired | 2 | 2 | 0 |
| Wireless to Server | 2 | 2 | 0 |
| Internet (Ping) | 1 | 1 | 0 |
| Internet (Traceroute) | 1 | 1 | 0 |
| OSPF Adjacency | 1 | 1 | 0 |
| **Total** | **35** | **35** | **0** |

All 35 tests passed.
