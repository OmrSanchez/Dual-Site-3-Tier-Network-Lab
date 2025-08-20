# Dual-Site-3-Tier-Network-Lab
A comprehensive GNS3 simulation of a resilient, dual-site enterprise network using a three-tier architecture. Implements OSPF, HSRP, NAT, and advanced Layer 2 security. 

# Enterprise Network Simulation in GNS3

## Project Overview

This project is a comprehensive simulation of a dual-site enterprise network built in GNS3. It employs a classic **three-tier hierarchical network design** (**Core**, **Distribution**, and **Access**) which serves as the industry-standard blueprint for building scalable, reliable, and manageable networks. The design separates network functions into distinct layers, which isolates failures, simplifies troubleshooting, and allows for modular growth.

The topology includes two distinct sites, "Local" and "Remote," simulating a main office and a branch office connected via a redundant core layer. This dual-site setup was designed to test inter-site routing, redundancy, and the application of consistent policies across a geographically dispersed enterprise.

The primary goal of this lab was to implement and validate a wide range of foundational and advanced networking technologies in an integrated environment. The project demonstrates a holistic understanding of how individual protocols and features work together to create a functional, secure, and resilient enterprise network.

### Core Technologies Implemented:
* **Switching:** VLANs, 802.1Q Trunks, VTP, EtherChannel (PAgP & LACP), Rapid PVST+
* **Routing:** OSPF, Static & Default Routing (IPv4 & IPv6)
* **Redundancy:** HSRP (First-Hop Redundancy Protocol), Redundant Links & Devices
* **Network Services:** DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH
* **Security:** Extended ACLs, Port Security, DHCP Snooping, Dynamic ARP Inspection (DAI)
* **WAN Technologies:** NAT (Static) & PAT (Dynamic Pool)

---

## Network Topology

The network is structured in a three-tier hierarchy:

1.  **Access Layer:** Provides endpoint connectivity to the network (e.g., PCs, servers). This layer is characterized by features like Port Security, DHCP Snooping, and STP port enhancements.
2.  **Distribution Layer:** This is the "workhorse" layer that aggregates traffic from the Access layer. It serves as the boundary between Layer 2 and Layer 3 domains, and is the primary point for routing, policy enforcement (ACLs), and first-hop redundancy (HSRP).
3.  **Core Layer:** The backbone of the network, providing high-speed, reliable transport between distribution layer devices and sites. This layer is engineered for speed and availability, focusing purely on switching packets as fast as possible without complex policy manipulation.



---

## Configuration Walkthrough & Skills Showcase

The network was configured in a phased approach to ensure stability and logical progression. Each part builds upon the last, from establishing basic connectivity to deploying advanced features.

### Part 1 & 2: Layer 2 Foundation & Redundancy

This initial phase focused on building a resilient and segmented Layer 2 fabric. A stable Layer 2 is the prerequisite for all subsequent Layer 3 functionality.

* **Initial Device Setup:** Basic configurations including **hostnames**, **user accounts**, and **console/enable passwords** were applied.
    * **Rationale:** This is the foundational step for network management, ensuring each device is identifiable, manageable, and secure from unauthorized local access.
* **VLANs & VTP:** **VLANs** were created to segment the network into separate broadcast domains. **VTP (VLAN Trunking Protocol)** was used in Server/Client mode to centralize and propagate the VLAN database.
    * **Rationale:** Segmentation improves security by isolating traffic and enhances performance by reducing the size of broadcast domains. VTP simplifies administration by ensuring VLAN consistency across the network, reducing the risk of manual configuration errors. However, it's used with caution in production due to the risk of "VTP bombing," where an incorrect configuration could wipe out the VLAN database.
* **EtherChannel (PAgP & LACP):** Link aggregation was configured between switches at all layers using both Cisco's proprietary **PAgP** (configured in `desirable` mode) and the industry-standard **LACP** (configured in `active` mode).
    * **Rationale:** EtherChannel bundles multiple physical links into a single logical channel. This provides **increased bandwidth** and **fault tolerance**; if one link in the bundle fails, traffic is automatically redirected over the remaining links with no disruption. Using `desirable` and `active` modes ensures that the switches actively attempt to negotiate a channel, which is a more robust configuration than passive modes.
* **802.1Q Trunks & Access Port Hardening:** Trunk ports were configured on inter-switch links to carry traffic for multiple VLANs. Access ports were manually configured, and all **unused ports were administratively shut down** and assigned to an unused VLAN.
    * **Rationale:** Trunks are essential for extending VLANs across multiple switches. Hardening unused ports is a fundamental security best practice to prevent unauthorized devices from connecting to the network and to mitigate attacks like VLAN hopping.

### Part 3: Layer 3 & First-Hop Redundancy

With a stable Layer 2 fabric, the next step was to enable inter-VLAN routing and ensure high availability for end-user devices.

* **IP Addressing & L3 EtherChannel:** IP addresses were assigned to **Switched Virtual Interfaces (SVIs)** on the distribution switches, making them the default gateways for their respective VLANs. The links between the distribution and core layers were configured as Layer 3 routed ports, and a Layer 3 EtherChannel was built between the core switches.
    * **Rationale:** SVIs allow a multilayer switch to route traffic between different VLANs. Using Layer 3 links to the core is a best practice that contains the Layer 2 domain to the access/distribution block, improving stability and scalability by preventing STP issues from impacting the entire network.
* **HSRP (Hot Standby Router Protocol):** HSRP was configured on the distribution switches for each VLAN. The active HSRP router was varied between the two distribution switches for different VLANs.
    * **Rationale:** HSRP provides **default gateway redundancy**. Two or more routers share a virtual IP address. If the active router fails, the standby router seamlessly takes over. By making DSW-1 active for some VLANs and DSW-2 active for others, we achieve **load balancing** in addition to redundancy under normal operating conditions.

### Part 4: Spanning Tree Protocol (STP) Optimization

STP is crucial for preventing Layer 2 loops. This phase focused on tuning STP for stability, fast convergence, and predictable traffic flow.

* **Rapid PVST+ & Root Bridge Placement:** The network was configured to use **Rapid Per-VLAN Spanning Tree Plus**. The primary root bridge for each VLAN was manually set to align with the active HSRP router for that VLAN, with the secondary root on the standby HSRP router.
    * **Rationale:** RPVST+ provides much faster network convergence (sub-second) after a topology change. Aligning the STP root with the HSRP active router is a critical design principle. It ensures that traffic takes the most direct path to its gateway, preventing suboptimal routing where frames have to cross the inter-distribution link to reach their gateway (an issue known as "traffic tromboning").
* **PortFast & BPDU Guard:** These features were enabled on all access-layer ports connected to end devices.
    * **Rationale:** **PortFast** allows ports to bypass the listening and learning states, bringing them up almost instantly for clients. **BPDU Guard** is a security feature that protects the STP domain by shutting down a port if it receives a BPDU from an unauthorized device (like a rogue switch).

### Part 5: Dynamic Routing with OSPF

This phase established dynamic routing, allowing the network to automatically learn paths and adapt to changes without manual intervention.

* **OSPF Configuration:** **OSPF (Open Shortest Path First)**, a link-state routing protocol, was configured on all Layer 3 devices. Interfaces were configured as `point-to-point` where appropriate, and LAN-facing SVIs were made `passive`.
    * **Rationale:** OSPF is highly scalable and converges quickly, making it the standard for interior routing in enterprise networks. Configuring point-to-point network types on dedicated links optimizes OSPF by disabling the Designated Router (DR) election process. Making interfaces passive prevents OSPF from sending hello packets into the access layer, which is unnecessary and a minor security risk.
* **Default Route Injection:** A static default route was configured on the R1 "edge" router pointing to the cloud, and this route was injected into the OSPF domain using the `default-information originate` command.
    * **Rationale:** This allows all internal routers to automatically learn the path to the internet. It's more scalable and manageable than configuring static default routes on every router in the network.

### Part 6: Core Network Services

A network is not functional without core services. This phase involved setting up DHCP, DNS, and other essential protocols for management and operation.

* **DHCP & DHCP Relay:** A central DHCP server was configured. The `ip helper-address` command was configured on the distribution switch SVIs.
    * **Rationale:** DHCP automates the assignment of IP addresses. Since DHCP requests are broadcasts and cannot cross routers, the **IP helper-address** command is essential for allowing a single, centralized server to service clients across multiple subnets.
* **DNS, NTP, SNMP, Syslog:** These critical network services were configured.
    * **Rationale:** **DNS** for name resolution, **NTP** for time synchronization (critical for log correlation), **SNMP** for network monitoring, and **Syslog** for centralized logging are all fundamental to a manageable network.
* **SSH, FTP, and NAT/PAT:**
    * **SSH** was enabled for encrypted, secure remote management.
    * **FTP** was used to demonstrate IOS upgrade procedures.
    * **Static NAT** was configured to map a public IP address to an internal server.
    * **Dynamic PAT (Overload)** was configured to translate many private internal addresses to a small pool of public addresses, conserving IPv4 space.

### Part 7: ACLs and Layer 2 Security

With the network fully functional, the focus shifted to implementing security policies to protect network resources.

* **Extended ACLs:** Access Control Lists were created and applied to filter traffic based on source/destination IP, protocol, and port number.
    * **Rationale:** ACLs are the primary tool for enforcing security policies. Extended ACLs were applied inbound on the distribution layer SVIs (as close to the source as possible) to filter traffic efficiently before it traverses the network core.
* **Port Security:** This was enabled on access ports to restrict access to only specific MAC addresses (`sticky` mode).
    * **Rationale:** This prevents unauthorized devices from connecting and mitigates MAC address spoofing attacks by locking a port to the MAC address(es) of legitimate devices.
* **DHCP Snooping & Dynamic ARP Inspection (DAI):**
    * **Rationale:** These are advanced security features that prevent common LAN attacks. **DHCP Snooping** builds a trusted binding table and stops rogue DHCP servers. **DAI** uses that table to validate ARP packets, preventing man-in-the-middle attacks that rely on ARP poisoning.

### Part 8: IPv6 Implementation

The final phases introduced forward-looking technology to demonstrate modern networking skills.

* **IPv6:** Basic IPv6 addressing (including EUI-64) and static default routing were configured on the core and edge devices.
    * **Rationale:** Demonstrates the ability to configure and operate a dual-stack (IPv4 and IPv6) network, a crucial skill as the internet continues its transition to IPv6.

---

## Recommended Images for Showcase

To visually prove and explain your configurations, I recommend capturing CLI output for the following key technologies.

1.  **HSRP Redundancy:**
    * **Command:** `show standby brief`
    * **Why:** This is the definitive command for verifying HSRP. It clearly shows the virtual IP, which router is `Active` and which is `Standby`, and their priorities. Show this from both distribution switches to highlight the active/standby roles and load balancing.
    * 

2.  **OSPF Routing:**
    * **Commands:** `show ip ospf neighbor` and `show ip route ospf`
    * **Why:** The first command proves that your OSPF adjacencies are correctly formed. The second command shows the specific routes learned via OSPF, confirming that your routing protocol is successfully propagating network reachability information.
    * 

3.  **NAT/PAT Translations:**
    * **Command:** `show ip nat translations`
    * **Why:** After generating traffic, this command displays the active translation table. It shows the mapping from the inside local address (private IP) to the inside global address (public IP), demonstrating that PAT is working as intended.
    * 

4.  **Layer 2 Security (DHCP Snooping & DAI):**
    * **Commands:** `show ip dhcp snooping binding` and `show ip arp inspection statistics`
    * **Why:** The first command displays the trusted MAC-to-IP address bindings, which is the database DAI uses. The second command shows statistics on how many ARP packets have been inspected, dropped, or forwarded, proving that DAI is actively protecting the network.
    * 

5.  **End-to-End Connectivity:**
    * **Action:** A `traceroute` from a PC in the "Remote" site to a server in the "Local" site.
    * **Why:** A successful traceroute is the ultimate proof that your entire network is integrated and functioning correctly. It shows the packet's path hop-by-hop through the access, distribution, and core layers, validating everything from Layer 2 forwarding to dynamic routing.
    *
