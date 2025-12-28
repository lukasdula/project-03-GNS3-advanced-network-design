
<br>

# **08 - Conclusion and Summary**

<br>

## **Final Evaluation**

This third GNS3 project defines an advanced routed and switched network focused on redundancy, availability, and operational verification. The design combines VLAN segmentation, inter-VLAN and dynamic routing, gateway redundancy, centralized services such as DHCP and NTP, and monitoring into a single system. The main goal is to demonstrate network reliability by testing features such as VRRP for gateway redundancy and LACP for link bundling, supported by loopback interfaces, Rapid Spanning Tree Protocol, and redundant paths across the switching and routing layers.

The topology uses two core routers (R1 and R2), multiple Layer 2 switches, an internal Xubuntu Server, an Xubuntu-Admin workstation, and user devices across several VLANs. The IPv4 addressing plan applies different subnet mask sizes based on network role and expected host count, improving clarity and scalability. VLAN 99 is used for centralized device management.

The switching layer includes redundancy and protection mechanisms such as Rapid Spanning Tree Protocol, PortFast, BPDU Guard, and LACP to prevent loops, protect access ports, and increase link resilience.

Layer 2 and Layer 3 connectivity is provided through VLANs, trunk links, and 802.1Q subinterfaces. OSPF ensures dynamic routing and consistent reachability, supported by loopback interfaces as stable router identifiers. Gateway redundancy is implemented using VRRP, where router R1 operates as the active gateway and router R2 remains in a backup role. If the active router becomes unavailable, the backup router automatically takes over the gateway role, allowing the network to continue operating without interruption.

Core services are centralized on the Xubuntu Server. DHCP supplies dynamic IPv4 addressing using relay, NTP provides consistent time synchronization, and SSH enables secure management access via the management VLAN. Wireshark monitoring validates correct protocol operation and traffic behavior.

VRRP failover testing verifies correct gateway behavior during router failure. During this testing, a real NAT/PAT issue is identified and resolved through troubleshooting. The results confirm that routing, gateway redundancy, and address translation continue to operate correctly during simulated failures.


<br>

## **Project Overview**

1. **Network Topology and Devices**  
    The network is designed with two core routers, multiple access and distribution switches, an internal server, an administrative workstation, and user endpoints. The topology supports redundancy and clear separation of infrastructure, management, and user roles.
    
2. **Addressing and VLAN Planning**  
    IPv4 addressing and VLAN segmentation define separate networks for servers, administration, users, management, and routed links. Different subnet mask sizes are selected based on network purpose and scalability requirements, ensuring efficient address usage and clear network structure.
    
3. **Basic Device Configuration**  
    Core configuration establishes hostnames, interface addressing, port descriptions, and baseline administrative settings required for stable network operation and clear documentation.
    
4. **Switching and VLAN Configuration**  
    VLANs are created on all switches, access and trunk ports are assigned, and redundancy is implemented at the distribution layer. Rapid Spanning Tree Protocol, PortFast, BPDU Guard, and LACP are applied to prevent loops, protect access ports, and provide resilient Layer 2 connectivity.
    
5. **Routing, VRRP, and Internet Connectivity**  
    Inter-VLAN routing is implemented on both routers using 802.1Q subinterfaces. OSPF provides dynamic routing with loopback interfaces as stable router identifiers. NAT/PAT enables Internet access, and VRRP delivers default gateway redundancy across all VLANs.
    
6. **Core Services and Monitoring**  
    Centralized DHCP, NTP, and SSH services are deployed on the Xubuntu Server. Wireshark monitoring validates routing behavior, protocol operation, and address translation under normal operating conditions.
    
7. **VRRP Failover Testing** and Troubleshooting
    Failover testing verifies gateway behavior when the active router R1 becomes unavailable and the backup router R2 takes over the master role. A NAT/PAT issue observed during failover is identified and resolved, confirming that VRRP, NAT/PAT, and routing continue to operate correctly.
    

<br>

---


<br>


**Back to project overview:** 