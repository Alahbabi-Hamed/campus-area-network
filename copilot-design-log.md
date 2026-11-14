# AI-Assisted Design Log

This document records six design interactions where I consulted the AI assistant during the planning, configuration, and troubleshooting of the campus network. Each entry includes what I asked, what the AI recommended, how I evaluated that recommendation, and what I actually implemented.

---

## Interaction 1: Campus Topology and Hierarchical Design

**Design Phase:** Initial network architecture planning

**What I Asked:**  
I described the three-building campus layout from the lab brief and asked the AI to suggest a hierarchical topology that would connect all three buildings, support inter-VLAN routing, and scale reasonably for future growth.

**What the AI Suggested:**  
The AI recommended a classic three-tier hierarchy with a core layer, distribution layer, and access layer. It proposed placing a single Layer 3 core switch (3560-24PS) in Building A to handle all inter-VLAN routing through SVIs. Buildings B and C would each get their own 3560-24PS as distribution switches, aggregating traffic from their floor-level access switches before sending it up to the core. Building A access switches would connect directly to the core since the core switch is physically in that building. Each floor in every building would have one 2960-24TT access switch.

**My Evaluation:**  
The three-tier approach made sense given the multi-building requirement and the number of VLANs involved. I did question whether Buildings B and C actually needed their own L3 switches at the distribution layer or if L2 switches would be enough. Since the core switch handles all inter-VLAN routing through its SVIs, the distribution switches in B and C don't need to perform routing themselves. They just trunk VLANs up to the core. I kept the 3560-24PS at distribution anyway because it gives me the option to add routing later if the network grows, and the lab requires showing a proper hierarchical design. Using 2960s at access made sense because those switches only need L2 functionality, VLAN assignment and trunking to the distribution or core layer.

**What I Implemented:**  
I followed the suggested topology with minor adjustments to port assignments based on what was physically available on each switch model. The 3560-24PS only has 2 GigabitEthernet ports and 24 FastEthernet ports, so I had to plan carefully which links used Gig ports (the inter-building backbone trunks) and which used FastEthernet (connections to access switches and the server switch).

---

## Interaction 2: VLSM Subnet Design

**Design Phase:** IPv4 addressing and VLAN planning

**What I Asked:**  
I gave the AI the campus address block (10.50.0.0/16) along with the device counts for each department from the lab brief, and asked it to calculate VLSM subnets that would fit each department without wasting address space.

**What the AI Suggested:**  
The AI produced a VLSM plan with 17 subnets. It sized each subnet based on the actual device count plus a growth margin. Larger departments like Computer Science (43 devices) got /26 subnets (62 usable hosts), mid-size ones like Math and Science (18 devices) got /27 subnets (30 usable hosts), and smaller groups like IT Sandbox (8 devices) got /28 subnets (14 usable hosts). It also allocated /26 blocks for wireless VLANs, /27 for the server VLAN, /29 for the DMZ, /28 for management VLANs, and /30 for all point-to-point links.

**My Evaluation:**  
I checked every subnet calculation manually. The host counts matched the formula (2^n - 2 usable hosts per subnet), and the growth margins were reasonable, roughly 30 to 50 percent above current need, which is standard practice for campus networks. I verified there were no overlapping address ranges by listing every subnet's network address and broadcast address side by side. The three management VLANs (120, 121, 122) share the 10.50.120.0/24 space as contiguous /28 blocks, which I confirmed don't overlap: .0-.15, .16-.31, and .32-.47.

One thing I caught was that the AI initially suggested putting the CORE-to-CAMPUS-FW1 link on GigabitEthernet0/2 of the core switch. I had to adjust this because Gig0/1 was already assigned to the Building B trunk, and I needed to be careful not to double-assign ports. I mapped out all physical port assignments on paper before committing to the addressing plan.

**What I Implemented:**  
I used the VLSM plan as calculated, with the port assignment corrections I identified. The full addressing table is documented in ip-addressing.md.

---

## Interaction 3: VLAN and Trunk Configuration Strategy

**Design Phase:** Switch configuration planning

**What I Asked:**  
I asked how to set up VLANs across all switches so that each department's traffic stays segmented, and how trunking should work between the access, distribution, and core layers.

**What the AI Suggested:**  
The AI recommended creating all 17 VLANs on the core switch and only the locally needed VLANs on each distribution and access switch. Trunk links between switches should use 802.1Q encapsulation and allow only the VLANs relevant to that link. Access ports connecting to PCs should be set to the appropriate departmental VLAN. For the server room switch, ports would be assigned to VLAN 90 (Servers) or VLAN 100 (DMZ).

The AI also recommended setting up SVIs (Switch Virtual Interfaces) on CORE-SW1 for every VLAN, with the .1 address in each subnet serving as the default gateway. It suggested enabling `ip routing` on the core switch to activate inter-VLAN routing.

**My Evaluation:**  
The SVI approach is cleaner than router-on-a-stick for this many VLANs. With 17 VLANs, using sub-interfaces on a router would create a bottleneck on a single physical link. Having the core switch do the routing through hardware SVIs is more efficient and simpler to manage.

I made sure the distribution switches (BLDGB-SW1, BLDGC-SW1) did not have SVIs for the data VLANs, because I wanted all routing to happen at the core. If I had SVIs on the distribution switches too, I could end up with routing loops or asymmetric paths. The distribution switches only need the VLANs in their database so they can pass tagged traffic through their trunks.

I also verified that `ip routing` was enabled on the core switch. Without it, the SVIs act as management interfaces only and don't actually route between VLANs.

**What I Implemented:**  
Followed the plan as described. All 17 VLANs created on CORE-SW1, SVIs configured with gateway IPs, `ip routing` enabled. Distribution and access switches got only their local VLANs. Trunk links configured on all inter-switch connections.

---

## Interaction 4: Routing, NAT, and Internet Connectivity

**Design Phase:** WAN edge and Internet access design

**What I Asked:**  
I asked the AI how to set up routing between the campus core, the firewall, the edge router, and the simulated ISP so that internal users can reach the Internet, and how NAT should be configured to translate private addresses.

**What the AI Suggested:**  
The AI proposed the following routing chain:

1. CORE-SW1 runs OSPF Area 0 with CAMPUS-FW1 to exchange internal routes dynamically. CORE-SW1 also has a static default route pointing to CAMPUS-FW1.
2. CAMPUS-FW1 runs OSPF Area 0 on its inside interface to learn campus routes, and uses `default-information originate` to advertise a default route back to the core. It has a static default route pointing to EDGE-RTR1. NAT overload (PAT) is configured to translate the entire 10.50.0.0/16 block using the outside interface address.
3. EDGE-RTR1 has a static route for 10.50.0.0/16 pointing back to CAMPUS-FW1 and a default or static route toward ISP-RTR1.
4. ISP-RTR1 has a static route for 10.50.0.0/16 (or just 203.0.113.0/30) pointing to EDGE-RTR1.

**My Evaluation:**  
Running OSPF between just two devices (CORE-SW1 and CAMPUS-FW1) might seem like overkill when a static route would work. But OSPF is a lab requirement, and it also means that if I add more internal routers later, they automatically learn routes without manual updates. I validated that the OSPF area assignments were consistent on both sides (both Area 0) and that the network statements covered the correct interfaces.

For NAT, I verified the ACL matched the campus supernet (permit 10.50.0.0 0.0.255.255) and that the inside and outside interfaces were correctly designated. I also confirmed that the default route chain was complete. A missing default route on any device in the chain would break Internet access from that point onward.

During testing, I actually discovered that CAMPUS-FW1 was missing its default route to EDGE-RTR1. Internal pings between VLANs worked fine, but pings to the ISP (203.0.113.1) failed. The traceroute from a PC stopped at the firewall. I added `ip route 0.0.0.0 0.0.0.0 10.50.200.14` on CAMPUS-FW1 and Internet access started working immediately after that.

**What I Implemented:**  
Everything as planned, with the missing default route fix applied during testing. OSPF adjacency confirmed with `show ip ospf neighbor` (FULL state). NAT translations verified. Traceroute from an internal PC to the ISP shows the correct 4-hop path: gateway SVI, firewall, edge router, ISP router.

---

## Interaction 5: Wireless Network Configuration

**Design Phase:** Wireless deployment and DHCP for wireless clients

**What I Asked:**  
I asked how to configure the wireless access points in Packet Tracer to provide a campus-wide SSID with WPA2 security, and how to make sure wireless clients receive DHCP addresses from the correct building-specific wireless VLAN.

**What the AI Suggested:**  
The AI recommended using AP-PT access points with SSID "CampusNet", WPA2-PSK authentication, and the passphrase "Campus2024!". For wireless laptops, it said to replace the default Ethernet module with a WPC300N wireless module. It suggested configuring the AP switch ports as trunk ports so the wireless VLAN traffic could be tagged and sent to the core for DHCP.

**My Evaluation:**  
The WPA2-PSK configuration was straightforward and correct. The trunk port recommendation for AP switch ports, though, turned out to be wrong for Packet Tracer's AP-PT device. During testing, all three wireless laptops failed to get DHCP addresses. Wired PCs on the same switches worked perfectly fine.

After investigating, I found that Packet Tracer's AP-PT does not tag wireless traffic with a VLAN ID. It sends frames untagged. When the AP port is configured as a trunk, untagged traffic lands on the native VLAN (VLAN 1), which has no DHCP pool. The fix was to change all AP ports from trunk mode to access mode on the correct wireless VLAN (111 for Building A, 112 for Building B, 113 for Building C). Once I made that change, wireless DHCP started working.

This was a case where the AI gave a technically reasonable suggestion for real-world enterprise APs (which do support VLAN tagging), but it didn't account for how Packet Tracer's simplified AP-PT device actually handles traffic. I had to test and correct based on what I observed.

**What I Implemented:**  
AP-PT access points configured with WPA2-PSK on all floors and the outdoor area. Switch ports connecting to APs set to access mode on the building's wireless VLAN. Laptops equipped with WPC300N modules. All three wireless laptops received DHCP addresses after the port mode correction.

---

## Interaction 6: Troubleshooting Building B DHCP Failure

**Design Phase:** Testing and verification

**What I Asked:**  
After the initial configuration was complete, all PCs in Buildings A and C received DHCP addresses normally, but both PCs in Building B kept failing. I asked the AI to help diagnose why DHCP wasn't working for Building B VLANs (30 and 40) while everything else on the network functioned correctly.

**What the AI Suggested:**  
The AI walked through a structured troubleshooting sequence:

1. Check trunk status on BLDGB-SW1. Trunks showed up and correct VLANs were allowed.
2. Check VLAN database. VLANs 30 and 40 existed on all relevant switches.
3. Check SVIs on CORE-SW1. VLAN 30 and 40 SVIs were up/up with correct IPs.
4. Check DHCP pools. Pools existed with correct network statements and exclusions.
5. Check the MAC address table on CORE-SW1. No MAC entries from Building B VLANs were appearing.

The absence of MAC entries on the core switch for Building B VLANs was the key clue. The AI suggested the physical cabling might be the issue.

**My Evaluation:**  
The troubleshooting sequence was logical and correct. Each step eliminated a possible cause. The MAC address table check was the turning point: if the core switch has no MAC entries from Building B, then Layer 2 traffic isn't reaching the core at all. That means either the cable is disconnected, connected to the wrong port, or the trunk is misconfigured at the physical level.

I traced the cables in Packet Tracer and found the actual problem. The cable from BLDGB-SW1 Gig0/1 was plugged into CORE-SW1 Gig0/2 instead of Gig0/1. Gig0/2 on the core switch was configured as a routed port for the uplink to CAMPUS-FW1, not as a trunk. So Building B trunk traffic was hitting a routed interface and getting dropped. The CAMPUS-FW1 cable was in Gig0/1, which was configured as the Building B trunk port.

The fix was simply swapping the two cables so that BLDGB-SW1 connected to CORE-SW1 Gig0/1 (the trunk) and CAMPUS-FW1 connected to CORE-SW1 Gig0/2 (the routed port). After the swap, Building B PCs received DHCP addresses immediately.

The AI didn't identify the cable swap directly. It guided me through the diagnostic steps, and I found the physical issue by checking the topology myself. You can't rely on the AI alone for something like this. It can suggest logical checks, but verifying physical connections requires actually inspecting the setup.

**What I Implemented:**  
Corrected the cable connections between CORE-SW1 and both BLDGB-SW1 and CAMPUS-FW1. Verified Building B DHCP was working. No configuration changes were needed, the problem was purely physical.

---

## Summary

Across these six interactions, the AI assistant was most useful for initial design planning (topology, subnetting, routing protocol selection) and for structured troubleshooting. I had to correct its recommendations in two cases though: the AP trunk port issue (Interaction 5) and the missing default route I caught during testing (Interaction 4). The cable swap problem (Interaction 6) required hands-on inspection that no AI tool could perform remotely. The AI did accelerate the design process, but every suggestion required validation against the actual Packet Tracer environment before I could trust it.
