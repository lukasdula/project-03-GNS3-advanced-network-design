
<br>

# **1 - Network Topology and Devices**


<br>

## **1.1 Introduction**

This first chapter introduces the network topology and overall design goals.

The topology focuses on high availability and redundancy. It shows how the network remains operational during device or link failures by using redundant paths and gateway redundancy.

The network is intentionally kept small to keep the design simple and easy to follow, while still showing how failover works in practice.

The topology is built in layers, with routing devices at the top, distribution switches in the middle, and access switches for end devices.

Each chapter builds on the previous one and explains how this design improves availability and stability, ending with practical failover testing.


<br>

## **1.2  Topology Diagram**

![](images/Pasted%20image%2020251216004902.png)


<br>

## **1.3  Network Zones**

This section describes the main parts of the topology and their roles. 

- **Internet and ISP**
    
    - Simulate external connectivity to the Internet.
        
- **Routing zone (R1, R2)**
    
    - Provides Layer 3 connectivity for internal VLANs and the upstream connection.
        
    - Uses gateway redundancy to keep the default gateway available during failures.
        
- **Distribution zone (SW1, SW2)**
    
    - Connects the routing and access layers.
        
    - Provides redundant Layer 2 paths between switches.
        
- **Access zone (SW3, SW4)**
    
    - Connects end devices to the network.
        
- **End devices**
    
    - Xubuntu-Server: server VLAN.
        
    - Xubuntu-Admin: admin VLAN.
        
    - Office: client VLAN.
        
    - Warehouse: client VLAN.


<br>

## **1.4 Used Devices**

| **Device**       | **Image / Type** |     **Interfaces** | **Access** | **Purpose**                                                               |
| ---------------- | ---------------- | -----------------: | ---------- | ------------------------------------------------------------------------- |
| Router R1        | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Primary routing device; inter-VLAN routing; uplink to ISP; VRRP.          |
| Router R2        | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Secondary routing device; inter-VLAN routing backup; VRRP; uplink to ISP. |
| ISP Router       | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Simulated upstream router and Internet access.                            |
| Switches SW1-SW4 | Cisco IOSv-L2    | 4x GigabitEthernet | Telnet     | Layer 2 switching; VLANs; trunk links; redundant paths.                   |
| Xubuntu-Server   | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Internal server for basic services (DHCP).                                |
| Xubuntu-Admin    | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Administration workstation for network management.                        |
| Office PC        | VPCS             |        1x Ethernet | Console    | Client device in the Office VLAN.                                         |
| Warehouse PC     | VPCS             |        1x Ethernet | Console    | Client device in the Warehouse VLAN.                                      |
| Internet Host    | VPCS             |        1x Ethernet | Console    | Simulated external host for Internet and NAT/PAT testing.                 |

<br>

## **1.5 Connection Overview**

| **Device** | **Interface**  | **Connected to** | **Peer Interface** | **Purpose**                                         |
| ---------- | -------------- | ---------------- | ------------------ | --------------------------------------------------- |
| ISP Router | Gi0/5          | R1               | Gi0/5              | WAN uplink between ISP and core router R1.          |
| ISP Router | Gi0/4          | R2               | Gi0/4              | WAN uplink between ISP and core router R2.          |
| ISP Router | Gi0/0          | Internet Host    | e0                 | Connection to simulated Internet.                   |
| R1         | Gi0/0          | R2               | Gi0/0              | Direct router-to-router link for redundancy.        |
| R1         | Gi0/1          | SW1              | Gi0/1              | Uplink from core router to distribution switch SW1. |
| R2         | Gi0/2          | SW2              | Gi0/2              | Uplink from core router to distribution switch SW2. |
| SW1        | Gi0/0<br>Gi0/2 | SW2              | Gi0/0<br>Gi0/1     | Inter-switch uplink (LACP) + loop prevention (RSTP) |
| SW1        | Gi0/3          | SW3              | Gi0/3              | Downlink to access switch SW3.                      |
| SW2        | Gi0/3          | SW4              | Gi0/3              | Downlink to access switch SW4.                      |
| SW3        | Gi0/1          | Xubuntu-Server   | Gi0/0              | Access link for internal server.                    |
| SW3        | Gi0/2          | Xubuntu-Admin    | Gi0/0              | Access link for admin workstation.                  |
| SW4        | Gi0/1          | Office PC        | e0                 | Access link for Office VLAN client.                 |
| SW4        | Gi0/2          | Warehouse PC     | e0                 | Access link for Warehouse VLAN client.              |

<br>

## **1.6 Conclusion**

This chapter introduced the network topology, device roles, and connection structure used in the project. The design focuses on high availability and redundancy by using multiple routing paths, redundant gateways, and a layered switch architecture. This structure provides a stable foundation for testing failover scenarios and traffic continuity during device or link failures.

The next chapter focuses on IP addressing and VLAN planning. It defines VLAN roles, IP subnets, and management addressing, and prepares the network for routing, gateway redundancy, and further configuration steps in the following chapters.

<br>

---

<br>


**Next chapter:**[Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
