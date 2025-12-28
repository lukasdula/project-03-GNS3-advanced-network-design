
<br>

# **2 - Addressing and VLAN Planning**

<br>

## **2.1 Introduction**

This second chapter defines the IPv4 addressing plan and VLAN layout for the advanced network design. The addressing scheme is designed to support redundancy, clear separation of network zones, and predictable failover behavior.

The addressing plan uses variable subnet sizes based on the role and expected size of each network segment. Larger subnets are allocated to client-facing VLANs to support higher device density and future growth, while smaller subnets are used for server, administrative, and management segments to keep these areas controlled and easy to manage.

Default gateways for internal VLANs are provided using virtual IP addresses on redundant routing devices. This approach ensures a stable gateway address for end devices and management infrastructure during router or link failures and prepares the network for further routing and redundancy configuration in the following chapters.


<br>

## **2.2 Topology Diagram**

![](images/Pasted%20image%2020251216183044.png)


<br>

## **2.3 IP Addressing and Subnet Overview**


| **Device / Interface**              | **Description**                     | **IP Address**                     | **Subnet Mask** | **Default Gateway**   | **Assigned Network** |
| ----------------------------------- | ----------------------------------- | ---------------------------------- | --------------- | --------------------- | -------------------- |
| ISP Router Gi0/5                    | ISP to R1 point-to-point link       | 192.0.2.1                          | 255.255.255.252 | N/A                   | 192.0.2.0/30         |
| Router R1 Gi0/5                     | R1 to ISP point-to-point link       | 192.0.2.2                          | 255.255.255.252 | N/A                   | 192.0.2.0/30         |
| ISP Router Gi0/4                    | ISP to R2 point-to-point link       | 192.0.2.5                          | 255.255.255.252 | N/A                   | 192.0.2.4/30         |
| Router R2 Gi0/4                     | R2 to ISP point-to-point link       | 192.0.2.6                          | 255.255.255.252 | N/A                   | 192.0.2.4/30         |
| ISP Router Gi0/0                    | ISP to simulated Internet host      | 100.64.10.1                        | 255.255.255.252 | N/A                   | 100.64.10.0/30       |
| Internet Host e0                    | Simulated external host             | 100.64.10.2                        | 255.255.255.252 | 100.64.10.1           | 100.64.10.0/30       |
| Router R1 Gi0/0 and Router R2 Gi0/0 | Inter-router point-to-point link    | R1: 10.90.200.1<br>R2: 10.90.200.2 | 255.255.255.252 | N/A                   | 10.90.200.0/30       |
| Xubuntu-Server                      | Internal server (static)            | 10.90.10.10                        | 255.255.255.192 | 10.90.10.1 (VRRP VIP) | 10.90.10.0/26        |
| Xubuntu-Admin                       | Administrative workstation (static) | 10.90.20.10                        | 255.255.255.192 | 10.90.20.1 (VRRP VIP) | 10.90.20.0/26        |
| Office PC                           | Office client (DHCP)                | 10.90.30.100-10.90.31.199          | 255.255.254.0   | 10.90.30.1 (VRRP VIP) | 10.90.30.0/23        |
| Warehouse PC                        | Warehouse client (DHCP)             | 10.90.40.100-10.90.41.199          | 255.255.254.0   | 10.90.40.1 (VRRP VIP) | 10.90.40.0/23        |
| Switches SW1-SW4 (VLAN 99 SVI)      | Switch management (static)          | 10.90.99.11-10.90.99.14            | 255.255.255.240 | 10.90.99.1 (VRRP VIP) | 10.90.99.0/28        |

>Notes: VRRP virtual IP addresses are used as default gateways to provide a stable gateway during router failover. Router interface IP addresses are defined separately and are used for routing and redundancy protocols. R1 operates as the primary VRRP gateway, while R2 provides backup gateway functionality.

<br>

## **2.4 VLAN Planning**

| VLAN ID | VLAN Name  | Description / Purpose       | Assigned Network | Default Gateway (VRRP VIP) | R1 Subinterface IP | R2 Subinterface IP | Assigned Devices   | Switch Assoc       |
| ------- | ---------- | --------------------------- | ---------------- | -------------------------- | ------------------ | ------------------ | ------------------ | ------------------ |
| 10      | Server     | Server infrastructure       | 10.90.10.0/26    | 10.90.10.1                 | 10.90.10.2         | 10.90.10.3         | Xubuntu-Server     | SW3                |
| 20      | Admin      | Administrative workstations | 10.90.20.0/26    | 10.90.20.1                 | 10.90.20.2         | 10.90.20.3         | Xubuntu-Admin      | SW3                |
| 30      | Office     | Office user workstations    | 10.90.30.0/23    | 10.90.30.1                 | 10.90.30.2         | 10.90.30.3         | Office PCs         | SW4                |
| 40      | Warehouse  | Warehouse user workstations | 10.90.40.0/23    | 10.90.40.1                 | 10.90.40.2         | 10.90.40.3         | Warehouse PCs      | SW4                |
| 99      | Management | Network device management   | 10.90.99.0/28    | 10.90.99.1                 | 10.90.99.2         | 10.90.99.3         | SW1, SW2, SW3, SW4 | SW1, SW2, SW3, SW4 |

<br>

## **2.5 Conclusion**

This chapter defined an IPv4 addressing and VLAN design that reflects common enterprise practices while maintaining clarity and scalability. Subnet sizes are selected based on the role of each network segment, allowing client VLANs to support growth while keeping server, administrative, and management areas controlled.

Virtual IP addresses are used as default gateways on redundant routing devices to provide stable connectivity during failures and to separate logical design from physical implementation. Overall, the addressing and VLAN structure prepares the network for stable routing and next redundancy and security configuration in the following chapters.

<br>


---

<br>

**Next chapter:**



