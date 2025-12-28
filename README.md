# **Advanced Network Design** 

<br>

## **Introduction and Objectives**

This project presents an advanced routed and switched network built in GNS3 as part of my study portfolio. The main goal is to demonstrate network redundancy, availability, and resistance to network failures in a realistic Layer 2 and Layer 3 design.

The network uses VLAN segmentation, inter-VLAN routing, dynamic routing with OSPF, and gateway redundancy with VRRP to ensure continuous connectivity during router failures. Redundant links are implemented using LACP, and Rapid Spanning Tree mechanisms protect the switching layer. Centralized services such as DHCP and NTP (Chrony on Xubuntu) support stable network operation and verification.

Security mechanisms are kept limited in this project to keep the focus on routing, redundancy, and failover operation. Network security is addressed in a separate project.

The topology uses a limited number of end-host VLANs for demonstration and design purposes. The design prioritizes clear validation of routing logic, gateway redundancy, failover testing, and monitoring rather than full enterprise user simulation.

<br>

## **Topology Diagram**

![](images/Pasted%20image%2020251228214908.png)










<br>

## **Network Zones**

The network is divided into clear functional zones:

- ISP Zone – simulated internet router for external connectivity
    
- Server Zone (VLAN 10) – Xubuntu Server providing centralized services
    
- Admin Zone (VLAN 20) – Xubuntu-Admin workstation for management and testing
    
- User Zones – Office (VLAN 30) and Warehouse (VLAN 40) clients used for validation
    
- Management Zone (VLAN 99) – device management and monitoring
    
- Routing and Switching Zone – core routers and switching infrastructure
    


<br>

## **Project Structure**

1. Network Topology and Devices
    
2. Addressing and VLAN Planning
    
3. Basic Device Configuration
    
4. Switching and VLAN Configuration
    
5. Routing, VRRP, and Internet Connectivity
    
6. Core Services and Monitoring
    
7. VRRP Failover Testing and Troubleshooting
    
8. Conclusion and Summary
    

<br>

## **Tools and Environment**

- **GNS3 version 2.2.54**
    
- **Wireshark Version 4.2.2**
    
- **Xubuntu VM** (kernel-based QEMU virtual machine inside GNS3)
    
- **Cisco IOSv Router**
    
    - _VIOS-ADVENTERPRISEK9-M, Version 15.9(3)M6_
        
- **Cisco IOSv-L2 Switch**
    
    - _vios_l2-ADVENTERPRISEK9-M, Version 15.2(20170321)_
        
- **Visual Studio Code** (documentation editing)
    
- **Obsidian** (notes, summaries and screenshots)
    


<br>

## **Key Project Features**

- VLAN segmentation with access and trunk ports
    
- Inter-VLAN routing using 802.1Q subinterfaces
    
- Link aggregation using LACP
    
- Rapid Spanning Tree with PortFast and BPDU Guard
    
- Dynamic routing with OSPF and loopback interfaces
    
- Gateway redundancy using VRRP
    
- NAT and PAT for simulated Internet access
    
- Centralized DHCP using relay
    
- Centralized NTP using Chrony on Xubuntu
    
- Secure management access using SSH
    
- Basic traffic monitoring with Wireshark
    
- Controlled VRRP failover testing
    
- Real troubleshooting of a NAT/PAT configuration issue
    

<br>

## Author’s Note

In this project, I focused on building a network design that remains operational during device failures. The main goal was to better understand redundancy and availability, which led me to work with VRRP for the first time and to test gateway failover during router failures.

This project also includes my first use of LACP, which helped me understand link redundancy and its role in improving network availability. A stronger focus was placed on physical topology choices and their impact on overall network design.

Security was intentionally left out of this project. The primary focus was on redundancy, VRRP, routing design, and practical implementation rather than security features.

Monitoring network traffic with Wireshark added valuable insight into how the network operates beyond configuration. Together with the previous two projects, this work provided a solid foundation and practical experience, allowing me to move forward toward a larger enterprise-style project that combines features from all earlier designs.

<br>


---

<br>


© 2025 – Lukáš Dula | Home Network Project & Portfolio