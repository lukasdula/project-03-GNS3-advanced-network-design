
<br>

# **5 - Routing, VRRP, and Internet Connectivity**

<br>

## **5.1 Introduction**

This fifth chapter defines the Layer 3 configuration of the advanced redundant network design. It builds on the existing VLAN, switching, and baseline setup and introduces inter-VLAN routing, gateway redundancy, dynamic internal routing, and controlled external connectivity.

Inter-VLAN routing is implemented using a router-on-a-stick design with tagged subinterfaces on redundant core routers. VRRP provides a stable default gateway for all internal VLANs, allowing seamless gateway failover during router or link failures.

Dynamic routing is configured using OSPF in area 0 to enable internal route exchange between core routers. External connectivity is achieved using a static default route toward the ISP and NAT with PAT, which allows internal private networks to access external resources.

<br>

## **5.2 Topology Diagram**

![](images/Pasted%20image%2020251224220419.png)

<br>

## **5.3 Objectives**



1. Configure management addressing on VLAN 99, including IP addresses and default gateway settings on all switches.
    
2. Configure inter-VLAN routing using a router-on-a-stick architecture with VLAN subinterfaces.
    
3. Implement VRRP to provide a stable and redundant default gateway for internal VLANs.
    
4. Configure OSPF in area 0 to enable internal routing between core routers.
    
5. Define static default routing toward the ISP to forward external traffic.
    
6. Configure NAT with PAT to allow internal private networks to access external resources.
    
7. Verify end-to-end connectivity using a simulated Internet host.


<br>

## **5.4 VLAN 99 - Management Configuration**


This first configuration section defines the management configuration for VLAN 99 on all switches in the topology. VLAN 99 is used as a dedicated management network to provide centralized access to network devices.

Each switch is assigned a static IP address within the VLAN 99 subnet. The default gateway for all switches is set to the VRRP virtual IP address, so management access remains available during router or link failures.

This management network supports controlled administration access (for example SSH from the admin workstation) without mixing management traffic with user VLAN traffic.

<br>

### **VLAN 99 Addressing Overview**

VLAN ID: 99  
Subnet: 10.90.99.0/28  
Default Gateway (VRRP VIP): 10.90.99.1

| **Device** | **Management IP Address** | **Subnet Mask**     | **Default Gateway** |
| ------ | --------------------- | --------------- | --------------- |
| SW1    | 10.90.99.11           | 255.255.255.240 | 10.90.99.1      |
| SW2    | 10.90.99.12           | 255.255.255.240 | 10.90.99.1      |
| SW3    | 10.90.99.13           | 255.255.255.240 | 10.90.99.1      |
| SW4    | 10.90.99.14           | 255.255.255.240 | 10.90.99.1      |

---

### **Switch Management Interface Configuration**

This subsection configures the management SVI (interface VLAN 99) on each switch.

The same configuration structure is applied on SW1, SW2, SW3, and SW4. Only the management IP address changes.

#### **Switch SW1**

```plaintext
enable
configure terminal
interface vlan 99
ip address 10.90.99.11 255.255.255.240
no shutdown
exit
ip default-gateway 10.90.99.1
end
write
```
![](images/Pasted%20image%2020251224222512.png)

#### **Switch SW2**

```plaintext
enable
configure terminal
interface vlan 99
ip address 10.90.99.12 255.255.255.240
no shutdown
exit
ip default-gateway 10.90.99.1
end
write
```
![](images/Pasted%20image%2020251224222539.png)

#### **Switch SW3**

```plaintext
enable
configure terminal
interface vlan 99
ip address 10.90.99.13 255.255.255.240
no shutdown
exit
ip default-gateway 10.90.99.1
end
write
```
![](images/Pasted%20image%2020251224222613.png)

#### **Switch SW4**

```plaintext
enable
configure terminal
interface vlan 99
ip address 10.90.99.14 255.255.255.240
no shutdown
exit
ip default-gateway 10.90.99.1
end
write
```
![](images/Pasted%20image%2020251224222640.png)

<br>

### **Verification and Diagnostics**

Verification is performed on all switches. The example below shows the verification workflow on SW1 only.

#### **Switch SW1 (Example)**

```plaintext
enable
show ip interface brief
show running-config | section interface Vlan99
show running-config | include ip default-gateway
```
![](images/Pasted%20image%2020251224222806.png)


### **Results**

All switches are configured with static management IP addresses in VLAN 99 and use the VRRP virtual IP address (10.90.99.1) as the default gateway. The VLAN 99 SVI is operational and management reachability to the virtual gateway is confirmed.

VLAN 99 provides a dedicated management network that supports controlled administration access (for example SSH from the admin workstation) and keeps management traffic separated from user VLAN traffic. The switching layer is now ready for VRRP, routing, and NAT configuration in the following sections.

<br>

## **5.5 Inter-VLAN Routing - Router-on-a-Stick Configuration**


This section configures inter-VLAN routing using a router-on-a-stick architecture on both core routers. VLAN traffic is carried over a trunk link to each router, and each VLAN is terminated on a dedicated subinterface.

Each subinterface uses 802.1Q encapsulation and a unique IP address within the VLAN subnet. The VRRP virtual IP address (VIP) is configured in the next section and is used as the default gateway for end devices.


### **Subinterface Overview**

| **VLAN ID** | **R1 Subinterface (Gi0/1)** | **R2 Subinterface (Gi0/2)** | **VRRP VIP (Gateway)** |
| ----------- | --------------------------- | --------------------------- | ---------------------- |
| 10          | Gi0/1.10                    | Gi0/2.10                    | 10.90.10.1             |
| 20          | Gi0/1.20                    | Gi0/2.20                    | 10.90.20.1             |
| 30          | Gi0/1.30                    | Gi0/2.30                    | 10.90.30.1             |
| 40          | Gi0/1.40                    | Gi0/2.40                    | 10.90.40.1             |
| 99          | Gi0/1.99                    | Gi0/2.99                    | 10.90.99.1             |


### **Router Subinterface Configuration**

#### Router R1 (Trunk Interface Gi0/1)

```plaintext
enable
configure terminal
interface GigabitEthernet0/1.10
encapsulation dot1Q 10
ip address 10.90.10.2 255.255.255.192
no shutdown
exit
interface Gi0/1.20
encapsulation dot1Q 20
ip address 10.90.20.2 255.255.255.192
no shutdown
exit
interface Gi0/1.30
encapsulation dot1Q 30
ip address 10.90.30.2 255.255.254.0
no shutdown
exit
interface Gi0/1.40
encapsulation dot1Q 40
ip address 10.90.40.2 255.255.254.0
no shutdown
exit
interface Gi0/1.99
encapsulation dot1Q 99
ip address 10.90.99.2 255.255.255.240
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224222843.png)

#### Router R2 (Trunk Interface Gi0/2)

```plaintext
enable
configure terminal
interface GigabitEthernet0/2.10
encapsulation dot1Q 10
ip address 10.90.10.3 255.255.255.192
no shutdown
exit
interface Gi0/2.20
encapsulation dot1Q 20
ip address 10.90.20.3 255.255.255.192
no shutdown
exit
interface Gi0/2.30
encapsulation dot1Q 30
ip address 10.90.30.3 255.255.254.0
no shutdown
exit
interface Gi0/2.40
encapsulation dot1Q 40
ip address 10.90.40.3 255.255.254.0
no shutdown
exit
interface Gi0/2.99
encapsulation dot1Q 99
ip address 10.90.99.3 255.255.255.240
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224222933.png)


### **Verification and Diagnostics**

Verification is performed on both routers. The example below shows the verification workflow on R1 only.

#### Router R1 (Example)

```plaintext
enable
show ip interface brief
```
![](images/Pasted%20image%2020251224223358.png)

### **Results**

All VLAN subinterfaces are configured and operational on both core routers. Each VLAN is terminated at Layer 3 on R1 and R2 and is prepared for VRRP gateway redundancy in the next section.

<br>

## **5.6 VRRP - Gateway Redundancy Configuration**


This section implements VRRP to provide a stable and redundant default gateway for all internal VLANs. End devices and switches use VRRP virtual IP addresses (VIPs) as their default gateways. If one router fails, the second router automatically takes over the gateway role as a backup.



### **VRRP Design Overview**

- VRRP is configured on both core routers on all VLAN subinterfaces.
    
- VRRP VIP addresses are used as the default gateways for VLANs 10, 20, 30, 40, and 99.
    
- R1 operates as the primary VRRP gateway (higher priority).
    
- R2 operates as the standby VRRP gateway (lower priority).
    
- Preemption is enabled on R1 to restore the primary gateway role after recovery.
    

**VRRP VIP (Default Gateway) Overview**

|VLAN ID|VRRP Group|VRRP VIP (Gateway)|R1 IP Address|R2 IP Address|
|---|---|---|---|---|
|10|10|10.90.10.1|10.90.10.2|10.90.10.3|
|20|20|10.90.20.1|10.90.20.2|10.90.20.3|
|30|30|10.90.30.1|10.90.30.2|10.90.30.3|
|40|40|10.90.40.1|10.90.40.2|10.90.40.3|
|99|99|10.90.99.1|10.90.99.2|10.90.99.3|

---

### **VRRP Configuration**

VRRP is configured on each VLAN subinterface. R1 is configured with a higher priority to act as the active gateway. R2 is configured with a lower priority to act as the standby gateway.

#### Router R1 (Primary)

```plaintext
enable
configure terminal
interface Gig0/1.10
vrrp 10 ip 10.90.10.1
vrrp 10 priority 110
vrrp 10 preempt
exit
interface Gi0/1.20
vrrp 20 ip 10.90.20.1
vrrp 20 priority 110
vrrp 20 preempt
exit
interface Gi0/1.30
vrrp 30 ip 10.90.30.1
vrrp 30 priority 110
vrrp 30 preempt
exit
interface Gi0/1.40
vrrp 40 ip 10.90.40.1
vrrp 40 priority 110
vrrp 40 preempt
exit
interface Gi0/1.99
vrrp 99 ip 10.90.99.1
vrrp 99 priority 110
vrrp 99 preempt
exit
end
write
```
![](images/Pasted%20image%2020251224223804.png)
#### Router R2 (Standby)

```plaintext
enable
configure terminal
interface GigabitEthernet0/2.10
vrrp 10 ip 10.90.10.1
vrrp 10 priority 100
exit
interface Gig0/2.20
vrrp 20 ip 10.90.20.1
vrrp 20 priority 100
exit
interface Gi0/2.30
vrrp 30 ip 10.90.30.1
vrrp 30 priority 100
exit
interface Gi0/2.40
vrrp 40 ip 10.90.40.1
vrrp 40 priority 100
exit
interface Gi0/2.99
vrrp 99 ip 10.90.99.1
vrrp 99 priority 100
exit
end
write
```
![](images/Pasted%20image%2020251224223852.png)


### **Verification and Diagnostics**

Verification confirms VRRP state, priority, and VIP ownership. The example below shows verification on R1 only.

#### Router R1 (Example)

```plaintext
enable
show vrrp brief
show vrrp
```
![](images/Pasted%20image%2020251224224031.png)


```
show vrrp
```
![](images/Pasted%20image%2020251224224137.png)

### **Results**

VRRP is configured on both core routers for VLANs 10, 20, 30, 40, and 99. The VRRP VIP addresses provide stable default gateways for end devices and switches. R1 operates as the active gateway under normal conditions, while R2 provides standby gateway redundancy to maintain connectivity during router or link failures.

<br>

## **5.7 OSPF - Internal Routing Configuration**


This section configures OSPF in area 0 to enable internal routing between the core routers. OSPF is used to dynamically exchange routes for all internal VLAN networks and the inter-router link.

Loopback interfaces are used as stable router IDs for OSPF. This keeps the router ID consistent even if physical links go down.

The OSPF domain operates only inside the internal network. External connectivity toward the ISP is handled separately using a static default route in a later section.


### **OSPF Design Overview**

- OSPF area: 0 (backbone)
    
- OSPF process ID: 1
    
- Advertised networks:
    
    - VLAN subnets (10, 20, 30, 40, 99)
        
    - Router-to-router link (R1 ↔ R2)
        
- Router IDs are explicitly defined using loopback interfaces for stability.
    

### **Loopback Interface Configuration**

Loopback interfaces are configured to provide stable and predictable OSPF router IDs. Unlike physical interfaces, loopbacks remain operational as long as the router is running, which prevents router ID changes during link failures or topology changes.

Using loopback-based router IDs improves OSPF stability and simplifies troubleshooting in this project.

#### Router R1

```plaintext
enable
configure terminal
interface Loopback0
ip address 1.1.1.1 255.255.255.255
no shutdown
end
write
```
![](images/Pasted%20image%2020251224224312.png)

#### Router R2

```plaintext
enable
configure terminal
interface Loopback0
ip address 2.2.2.2 255.255.255.255
no shutdown
end
write
```
![](images/Pasted%20image%2020251224224344.png)


### **OSPF Configuration**

OSPF is enabled on both routers. All internal networks are advertised into area 0.

#### Router R1

```plaintext
enable
configure terminal
router ospf 1
router-id 1.1.1.1
network 10.90.10.0 0.0.0.63 area 0
network 10.90.20.0 0.0.0.63 area 0
network 10.90.30.0 0.0.1.255 area 0
network 10.90.40.0 0.0.1.255 area 0
network 10.90.99.0 0.0.0.15 area 0
network 10.90.200.0 0.0.0.3 area 0
end
write
```
![](images/Pasted%20image%2020251224224417.png)

#### Router R2

```plaintext
enable
configure terminal
router ospf 1
router-id 2.2.2.2
network 10.90.10.0 0.0.0.63 area 0
network 10.90.20.0 0.0.0.63 area 0
network 10.90.30.0 0.0.1.255 area 0
network 10.90.40.0 0.0.1.255 area 0
network 10.90.99.0 0.0.0.15 area 0
network 10.90.200.0 0.0.0.3 area 0
end
write
```
![](images/Pasted%20image%2020251224224542.png)


### **Verification and Diagnostics**

Verification confirms OSPF adjacency and route propagation. The example below shows the verification workflow on R1 only.

#### Router R1 (Example)

```plaintext
enable
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```
![](images/Pasted%20image%2020251224224951.png)


### **Results**

OSPF adjacency is successfully established between R1 and R2 in area 0. All internal VLAN networks and the inter-router link are dynamically exchanged. The internal routing domain is now complete and ready for static default routing and NAT configuration toward the ISP.

<br>

## **5.8 Static Default Routing Toward the ISP**


This section defines static default routing toward the ISP to enable external traffic forwarding. While OSPF is used for internal routing, external networks are not part of the OSPF domain and are reached using a static default route.

The default route directs all unknown destinations toward the ISP-facing interface. This approach clearly separates internal dynamic routing from external connectivity.


### **Static Default Route Configuration**

Static default routes are configured on both core routers. Each router forwards external traffic toward its respective ISP-facing next-hop address.

#### Router R1

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 192.0.2.1
end
write
```
![](images/Pasted%20image%2020251224225120.png)

#### Router R2

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 192.0.2.5
end
write
```
![](images/Pasted%20image%2020251224225144.png)


### **Verification and Diagnostics**

Verification confirms that the default route is installed and used for external destinations. The example below shows verification on R1 only.

#### Router R1 (Example)

```plaintext
enable
show ip route
show ip route static
```
![](images/Pasted%20image%2020251224225331.png)


### **Results**

Static default routing is successfully configured on both core routers. External traffic is forwarded toward the ISP using the default route, while internal routing remains fully managed by OSPF. The routing design now supports controlled and predictable external connectivity.


<br>

## **5.9 NAT and PAT Configuration**



This section configures NAT with PAT to allow internal private networks to access external resources through the ISP connection. All internal VLANs are allowed to use NAT/PAT.


### **Design Overview**

- NAT with PAT (NAT overload) is used for outbound traffic.
    
- Internal private networks are translated to a single ISP-facing address using port multiplexing.
    
- VLANs 10, 20, 30, 40 are included in the NAT policy.
    
- NAT is configured on both core routers for redundancy.
    
- The ISP router is not configured in this project. NAT is applied only on R1 and R2 at the ISP-facing edge.
    


### **NAT and PAT Configuration**

An access list defines which internal networks are allowed to use NAT. NAT overload is then applied using the ISP-facing (outside) interface.

Interface roles used in this project:

- R**1 inside:** Gi0/1 VLAN subinterfaces 
    
- R1 outside: Gi0/5 (toward ISP)
    
- **R2 inside:** Gi0/2 VLAN subinterfaces
    
- R2 outside: Gi0/4 (toward ISP)
    
* PAT overload translates internal traffic using the router’s external address.


#### Router R1

```plaintext
enable
configure terminal
access-list 1 permit 10.90.10.0 0.0.0.63
access-list 1 permit 10.90.20.0 0.0.0.63
access-list 1 permit 10.90.30.0 0.0.1.255
access-list 1 permit 10.90.40.0 0.0.1.255
interface GigabitEthernet0/1.10
ip nat inside
exit
interface GigabitEthernet0/1.20
ip nat inside
exit
interface GigabitEthernet0/1.30
ip nat inside
exit
interface GigabitEthernet0/1.40
ip nat inside
exit
interface GigabitEthernet0/5
ip nat outside
exit
ip nat inside source list 1 interface GigabitEthernet0/5 overload
end
write
```
![](images/Pasted%20image%2020251228003801.png)

#### Router R2

```plaintext
enable
configure terminal
access-list 1 permit 10.90.10.0 0.0.0.63
access-list 1 permit 10.90.20.0 0.0.0.63
access-list 1 permit 10.90.30.0 0.0.1.255
access-list 1 permit 10.90.40.0 0.0.1.255
interface GigabitEthernet0/2.10
ip nat inside
exit
interface GigabitEthernet0/2.20
ip nat inside
exit
interface GigabitEthernet0/2.30
ip nat inside
exit
interface GigabitEthernet0/2.40
ip nat inside
exit
interface Gi0/4
ip nat outside
exit
ip nat inside source list 1 interface GigabitEthernet0/4 overload
end
write
```
![](images/Pasted%20image%2020251228003823.png)


>**Note.:** The management VLAN (VLAN 99) is intentionally excluded from NAT/PAT configuration.  
 Management traffic is internal and must remain non-translated to allow reliable administrative access to network devices.
 In production environments, remote administration is typically performed via VPN, not through NAT.  
 Excluding VLAN 99 from NAT prevents interference with inter-VLAN management traffic and improves overall network security.


<br>

### **Final NAT/PAT Address Translation Overview**

This section summarizes how internal devices are represented on the external network through PAT. It shows the relationship between private internal addresses, their translated public form, and the external host accessed by internal VLAN clients.

| **Term**       | **Example Value** | **Meaning**                          | **Explanation**                                                                                                                                                                       |
| -------------- | ----------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inside Local   | 10.90.30.10       | Internal private address             | The actual IPv4 address of an internal device located inside the company network. This address exists only within the internal LAN.                                                   |
| Inside Global  | 192.0.2.2         | Public address for inside            | The public IPv4 address assigned to the ISP-facing interface of R1. All internal devices are translated to this address when accessing external networks using PAT overload.          |
| Outside Local  | 100.64.10.2       | External host as seen from inside    | The IPv4 address of the simulated Internet host as it appears from the internal network. In this design, it matches the outside global address.                                       |
| Outside Global | 100.64.10.2       | Real public address of external host | The actual IPv4 address of the simulated Internet host on the external network. This address is globally reachable and represents the external resource accessed by internal clients. |

### **Results**

NAT with PAT is configured on both core routers. Internal VLANs can access external resources through the ISP connection while internal addressing remains private.

<br>

## **5.10 Final Verification**


This final section verifies the complete Layer 3 design of the network. The verification focuses on real traffic flow, NAT/PAT operation, and basic routing status on the active core router.


### **Traffic Generation (Connectivity Test)**

To trigger NAT/PAT translations, traffic must be generated from an internal VLAN.

**Connectivity test description:**

- Traffic is sent from a device in VLAN 10 toward the simulated Internet host.
    
- The packet exits the network via the active VRRP gateway and the ISP-facing interface.
    

#### VLAN 10 Device to Internet Host

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251224233857.png)

A successful ping confirms that traffic is forwarded toward the ISP and NAT/PAT translation is triggered.


### **NAT/PAT Verification (Active Router)**

After traffic is generated, NAT/PAT operation is verified on the active VRRP gateway (R1).

#### Router R1 (Example)

```plaintext
enable
show ip nat translations
show ip nat statistics
```
![](images/Pasted%20image%2020251224234016.png)

The output should display active translations using the public IP address of R1.

---

### **Interface and Routing Status**

Basic interface and routing status is verified on the active core router.

#### Router R1 (Example)

```plaintext
enable
show ip interface brief
show ip ospf neighbor
```
![](images/Pasted%20image%2020251224234243.png)

To confirm basic external reachability from the router itself, an additional test ping is performed.

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251224234329.png)



### **Results**

End-to-end connectivity is successfully verified. Traffic from VLAN 10 reaches the simulated Internet host using NAT/PAT. Routing, VRRP gateway selection, and OSPF neighbor relationships operate as expected across the network.


<br>

## **5.11 Conclusion** 

This chapter defines the Layer 3 behavior of the network, including inter-VLAN routing, gateway redundancy, and dynamic internal routing. VRRP ensures a stable default gateway, while OSPF provides consistent routing between core routers.

External access is enabled using a static default route and NAT with PAT for internal VLANs, including the management network. End-to-end verification confirms correct traffic flow, and the design is ready for DHCP services and monitoring.




---

<br>


**Next chapter:** [Core Services and Monitoring](06-core-services-and-monitoring.md)

