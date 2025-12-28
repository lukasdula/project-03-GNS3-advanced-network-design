
<br>

# **6 - Core Services and Monitoring**

<br>

## **6.1 Introduction**

This is the sixth chapter of the project and it introduces core network services and basic monitoring. After completing switching, routing, redundancy, and Internet connectivity, the network is ready to support centralized services and secure management access.

In this part, DHCP, NTP, and SSH are introduced in a logical order. DHCP is used to automatically assign IP configuration to client devices across multiple VLANs, which confirms inter-VLAN routing, VRRP gateway operation, and DHCP relay. NTP is deployed on the Xubuntu Server to provide consistent time synchronization, which is important for logging, troubleshooting, and event checking in a redundant network.

The chapter also demonstrates secure management access using SSH from the Xubuntu Admin workstation to network switches via the management VLAN. Basic traffic monitoring is introduced using Wireshark in GNS3, focusing on ARP, OSPF, and NAT/PAT traffic. More advanced monitoring scenarios, such as VRRP failover analysis, are left for later to later chapters.

<br>

## **6.2 Topology Diagram**

![](images/Pasted%20image%2020251228030503.png)

<br>


### **6.3 Objectives**

1. Configure centralized DHCP on the Xubuntu Server for client VLANs and VLAN Connectivity Diagnostics
    
2. Configure NTP on the Xubuntu Server and configure NTP clients on routers and switches.
    
3. Enable SSH management access from Xubuntu Admin to switches and routers.
    
4. Monitor selected network protocols (ARP, OSPF, NAT/PAT) using Wireshark.


<br>

## **6.4 DHCP Services**

This section configures DHCP for dynamic clients in the network. The DHCP service runs on the Xubuntu Server in VLAN 10 and provides IPv4 addressing to client VLANs. Infrastructure hosts keep static IPv4 addressing.

DHCP requests are broadcast traffic and cannot cross VLAN boundaries by default. DHCP relay is used on the core routers to forward DHCP requests from client VLANs to the DHCP server.

### DHCP Scope Design

| **VLAN** | **VLAN Name** | **Subnet**    | **Gateway (VRRP)** | **DHCP Range**              | **Notes**       |
| -------: | ------------- | ------------- | ------------------ | --------------------------- | --------------- |
|       30 | Office        | 10.90.30.0/23 | 10.90.30.1         | 10.90.30.100 - 10.90.31.250 | Dynamic clients |
|       40 | Warehouse     | 10.90.40.0/23 | 10.90.40.1         | 10.90.40.100 - 10.90.41.250 | Dynamic clients |


---

### **DHCP Server Installation**

Install the DHCP service on Xubuntu Server.

```
sudo apt update
sudo apt install isc-dhcp-server -y
```
![](images/Pasted%20image%2020251227204508.png)

Bind the service to the correct interface.

```plaintext
sudo nano /etc/default/isc-dhcp-server
```


Set the interface line.

```plaintext
INTERFACESv4="ens3"
```
![](images/Pasted%20image%2020251227204902.png)

Enable and restart the service.

```plaintext
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
![](images/Pasted%20image%2020251227205041.png)

### **Default Route on DHCP Server**

The DHCP server needs a default route to reach routed VLANs.

```plaintext
sudo ip route add default via 10.90.10.1
```
![](images/Pasted%20image%2020251227205106.png)

### **DHCP Pool Configuration**

Edit the DHCP configuration file.

```plaintext
sudo nano /etc/dhcp/dhcpd.conf
```



**Example configuration.**

```conf
authoritative;
default-lease-time 3600;
max-lease-time 86400;

# Global options
option domain-name-servers 10.90.10.10;

# VLAN 10 - Server subnet (no DHCP pool)
subnet 10.90.10.0 netmask 255.255.255.192 {
}

# VLAN 30 - Office
subnet 10.90.30.0 netmask 255.255.254.0 {
 option routers 10.90.30.1;
 range 10.90.30.100 10.90.31.250;
}

# VLAN 40 - Warehouse
subnet 10.90.40.0 netmask 255.255.254.0 {
 option routers 10.90.40.1;
 range 10.90.40.100 10.90.41.250;
}
```
![](images/Pasted%20image%2020251227210827.png)


**Restart the DHCP service.**

```plaintext
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
![](images/Pasted%20image%2020251227210854.png)

<br>

### **DHCP Relay Configuration**

Configure DHCP relay on the core routers for client VLANs.

#### **Router R1**

```plaintext
enable
configure terminal
interface GigabitEthernet0/1.30
ip helper-address 10.90.10.10
exit
interface GigabitEthernet0/1.40
ip helper-address 10.90.10.10
exit
end
write
```
![](images/Pasted%20image%2020251227212050.png)

#### **Router R2**

```plaintext
enable
configure terminal
interface GigabitEthernet0/2.30
ip helper-address 10.90.10.10
exit
interface GigabitEthernet0/2.40
ip helper-address 10.90.10.10
exit
end
write
```
![](images/Pasted%20image%2020251227212101.png)


### **Results**

The DHCP service successfully provides IPv4 addresses to client devices in VLAN 30 and VLAN 40. Clients receive correct IP addressing, default gateway information via VRRP, and network connectivity across routed VLANs. DHCP relay operates correctly on the core routers, confirming proper integration of DHCP with inter-VLAN routing and the network design.


<br>

### **Verification and Ping Connectivity**

Set clients to use DHCP and confirm assigned settings.

#### Example - VPC client Office

```plaintext
ip dhcp
show ip
```
![](images/Pasted%20image%2020251227212228.png)

#### Xubuntu client

```bash
ip a
ip route
```
![](images/Pasted%20image%2020251227212355.png)

### **Diagnostics**

#### Xubuntu Server

```plaintext
sudo systemctl status isc-dhcp-server --no-pager -l
sudo tail -n 50 /var/log/syslog
```
![](images/Pasted%20image%2020251227212551.png)

#### Routers

```plaintext
show ip interface brief
show running-config | include helper-address
```
![](images/Pasted%20image%2020251227212642.png)

<br>

### **Ping Connectivity Verification**

The tests focus on client-to-server, client-to-admin, and infrastructure reachability. Successful results confirm that DHCP, inter-VLAN routing, and gateway configuration operate correctly.


#### **VLAN Endpoint Address Table**

| Device                  | VLAN | Role       | IPv4 address | Addressing |
| ----------------------- | ---: | ---------- | ------------ | ---------- |
| Xubuntu-Server          |   10 | Server     | 10.90.10.10  | Static     |
| Xubuntu-Admin           |   20 | Admin      | 10.90.20.10  | Static     |
| Office PC               |   30 | Office     | 10.90.30.100 | DHCP       |
| Warehouse PC            |   40 | Warehouse  | 10.90.40.100 | DHCP       |
| Switch Management (SVI) |   99 | Management | 10.90.99.12  | Static     |

#### **Ping Tests - Xubuntu Admin**

Xubuntu Admin verifies connectivity to client VLANs.

```plaintext
ping 10.90.30.100
ping 10.90.40.100
```
![](images/Pasted%20image%2020251227213933.png)

Result:

- ICMP replies received from VLAN 30 and VLAN 40 clients
    
- Inter-VLAN routing operates correctly
    



#### **Ping Tests - VLAN 30 (Office Client)**

Office client verifies connectivity to server and admin VLANs.

```plaintext
ping 10.90.10.10
ping 10.90.20.10
```
![](images/Pasted%20image%2020251227214654.png)

Result:

- Server and Admin hosts reachable from VLAN 30


#### **Ping Tests - VLAN 40 (Warehouse Client)**

Warehouse client verifies connectivity to infrastructure devices.

```plaintext
ping 10.90.10.10
ping 10.90.20.10
```
![](images/Pasted%20image%2020251227214805.png)

Result:

- Server, Admin reachable

### Ping Tests - Router R1

Router R1 verifies connectivity to administrative, client, and management endpoints.

```plaintext
ping 10.90.20.10
ping 10.90.40.100
ping 10.90.99.12
```
![](images/Pasted%20image%2020251227215013.png)

Result:

- All ICMP echo requests from Router R1 were successful.   


### **Verficication Results**

All ping tests completed successfully across VLAN 20, VLAN 30, VLAN 40, VLAN 10, and VLAN 99. Clients received correct IP addressing via DHCP, used proper default gateways, and communicated across routed VLANs without packet loss. The results confirm correct integration of DHCP, inter-VLAN routing, and core network services.

<br>

## **6.5 NTP Services**


This section configures time synchronization in the network using NTP. Accurate and consistent time is important for log analysis, troubleshooting, and correlation of network events across multiple devices.

The Xubuntu Server acts as a centralized NTP server. Network devices synchronize their time with this server to ensure that logs, routing events, and monitoring data use the same time reference.


### **Xubuntu Server - NTP Server (Chrony)**

Chrony is used as the NTP implementation on the Xubuntu Server. It provides reliable time synchronization and is well suited for virtualized environments.

#### **Installation**

```plaintext
sudo apt update
sudo apt install chrony -y
```
![](images/Pasted%20image%2020251227221632.png)

#### **Basic Configuration**

Edit the Chrony configuration file.

```plaintext
sudo nano /etc/chrony/chrony.conf
```


Ensure that the server allows NTP clients from internal networks and uses available time sources.

```plaintext
allow 10.90.0.0/16
```
![](images/Pasted%20image%2020251227222109.png)

**Restart and enable the service.**

```plaintext
sudo systemctl enable chrony
sudo systemctl restart chrony
sudo systemctl status chrony
```
![](images/Pasted%20image%2020251227222209.png)

### **Verification**

Check synchronization status on the server.

```plaintext
chronyc tracking
chronyc sources

```
![](images/Pasted%20image%2020251227222244.png)

### **Result:**

- Chrony service is running
    
- The server maintains a stable system time
    

---

### **Router R2 - NTP Client Example**

Router R2 is configured as an NTP client to synchronize its time with the Xubuntu Server.

#### **Configuration**

```plaintext
enable
configure terminal
ntp server 10.90.10.10 prefer
end
write
```
![](images/Pasted%20image%2020251227222936.png)


### **Router R2 - NTP Source Interface**

The router is configured to use its VLAN 10 subinterface as the NTP source. This interface belongs to the same network as the NTP server and provides a stable and reachable source address.

### Configuration

```
configure terminal
ntp source GigabitEthernet0/2.10
end
write
```
![](images/Pasted%20image%2020251228012721.png)

#### **Verification**

```plaintext
show ntp status
show clock
```
![](images/Pasted%20image%2020251228012812.png)

### **Result:**

NTP synchronization is verified on router R2. The device is synchronized to the internal NTP server and operates in a stable controlled state.
    

### **Final Results**

Time synchronization is successfully established using Chrony on the Xubuntu Server. Router R2 synchronizes its clock with the centralized NTP server, ensuring consistent timestamps across network devices. This configuration provides a reliable foundation for logging, troubleshooting, and future monitoring scenarios.


<br>

## **6.6 SSH Management Access**

This section demonstrates secure remote management of network devices using SSH. Instead of accessing switches directly from an end-host workstation, administrative access is performed via the router, which represents a realistic enterprise design where routers or firewalls act as controlled entry points to the management network.

The goal is to demonstrate secure administrative access from Router R1 to Switch SW4 using encrypted SSH sessions over the management VLAN.


### **Switch SW4 - SSH Configuration**

The switch is configured to allow secure SSH access on its management interface in VLAN 99. Local user authentication is used, and insecure protocols such as Telnet are explicitly disabled.

#### **Configuration:**

```plaintext
enable
configure terminal
ip domain-name project.local
username netadmin privilege 15 secret Sz2kystsLK54s
crypto key generate rsa
2048
ip ssh version 2
line vty 0 4
login local
transport input ssh
exit
end
write
```
![](images/Pasted%20image%2020251228012606.png)


### **Router R1 - SSH Client Access**

Router R1 acts as the administrative access point. In real deployments, administrators typically connect to the router via VPN or secure management access and then initiate SSH sessions to switches within the management VLAN.

No additional configuration is required on R1 for SSH client functionality, as the SSH client is available by default.


### **SSH Access Test**

From Router R1, an SSH connection is initiated to the management IP address of Switch SW4.

```plaintext
ssh -l netadmin 10.90.99.14
```
![](images/Pasted%20image%2020251228012323.png)

### **Verification**

On the switch, active SSH sessions can be verified.

```plaintext
show ssh
show users
```
![](images/Pasted%20image%2020251228012417.png)


## **Results**

Secure SSH management access to Switch SW4 is successfully established from Router R1. Remote administration is performed using encrypted communication, reflecting a realistic and secure enterprise management design where management VLAN access is restricted and mediated by routing infrastructure.

<br>

## **6.7 Wireshark Monitoring**


This final section demonstrates basic traffic monitoring using Wireshark in GNS3. The goal is to observe selected control-plane and data-plane protocols to verify correct network behavior after routing, redundancy, and NAT/PAT configuration.

Monitoring is performed directly on links between devices using the built-in packet capture feature in GNS3. This approach reflects real-world troubleshooting, where engineers analyze live traffic to validate connectivity and protocol operation.

### Objectives

- Observe ARP address resolution inside internal VLANs
    
- Monitor OSPF routing protocol traffic between core routers
    
- Verify NAT/PAT address translation on the WAN edge
    


### **ARP Traffic Monitoring**

**Capture location**

- Link between Xubuntu-Admin and SW3 (Access VLAN 20)
    

**Trigger**

- Ping from Xubuntu-Admin to default gateway (VRRP VIP)
    

```plaintext
ping 10.90.20.1
```
![](images/Pasted%20image%2020251228015112.png)

**Wireshark filter**

```plaintext
arp
```
![](images/Pasted%20image%2020251228015047.png)

**Observation**

- ARP Request is sent by the client to resolve the MAC address of the default gateway
    
- ARP Reply is received from the router providing the VRRP virtual IP
    

### **Result**  

ARP monitoring confirms correct VRRP operation in VLAN 20. The Xubuntu Admin host resolves the default gateway IP (10.90.20.1) to the VRRP virtual MAC address (00:00:5e:00:01:14), demonstrating proper gateway redundancy and Layer 2 to Layer 3 interaction.

---

### **OSPF Traffic Monitoring**

**Capture location**

- Routed link between R1 and R2
    

**Wireshark filter**

```plaintext
ospf
```
![](images/Pasted%20image%2020251228015828.png)

**Observation**

- OSPF Hello packets are exchanged between R1 and R2
    
- Multicast addresses 224.0.0.5 and 224.0.0.6 are used
    

### **Result**  

OSPF monitoring confirms correct routing adjacency between core routers. Regular OSPF Hello packets are exchanged between R1 (10.90.200.1) and R2 (10.90.200.2) using the multicast address 224.0.0.5, indicating a stable OSPF neighbor relationship and proper operation of the routing control plane.

---

### **NAT/PAT Traffic Monitoring**

**Capture location**

- WAN link between R1 and the ISP router
    

**Trigger**

- Ping from an internal client to the simulated Internet host
    

```plaintext
ping 100.64.10.1
```
![](images/Pasted%20image%2020251228020322.png)

**Wireshark filter**

```plaintext
icmp
```
![](images/Pasted%20image%2020251228020307.png)

**Observation**

- ICMP packets leaving the router use the public IP address
    
- Source IP translation confirms PAT overload operation
    

### **Result**  

ICMP monitoring confirms correct NAT/PAT operation on the WAN edge. Internal traffic originating from the private address 192.0.2.2 is translated to the public IP address 100.64.10.1 before being forwarded toward the Internet host. ICMP echo replies are successfully returned to the translated address, verifying bidirectional connectivity and proper address translation for outbound traffic.

### **Summary**

Wireshark monitoring confirms correct operation of ARP, OSPF, and NAT/PAT within the network. Captured traffic validates proper Layer 2 communication, dynamic routing between core devices, and secure address translation at the network edge.


<br>


## **6.8 Conclusion**

The results confirm that core services and monitoring operate as expected.  
Clients receive correct IP configuration via DHCP, devices share a consistent time source using NTP, and secure management access is available through SSH. Traffic captures verify correct Layer 2 resolution, stable OSPF adjacencies, and proper NAT/PAT address translation on the WAN edge.

The next chapter focuses on testing VRRP failover behavior during router outages and includes targeted troubleshooting scenarios to validate redundancy under failure conditions.

<br>


---

<br>


**Next chapter:** [VRRP Failover Testing and Troubleshooting](07-VRRP-failover-testing-and-troubleshooting.md)
