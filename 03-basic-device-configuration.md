
<br>

# **3 - Basic Device Configuration**


<br>

## **3.1 Introduction**

This section defines the basic device configuration for the advanced network topology. The configuration establishes a clean and stable baseline before introducing VLANs, gateway redundancy, and dynamic routing in later chapters.

Static IPv4 addressing is applied to the Xubuntu Server and Xubuntu Admin workstation to ensure predictable connectivity for management and infrastructure services. All routers and switches receive consistent hostnames to support clear device identification and operational clarity.

IPv4 addressing is configured on all point-to-point links between routing devices, including the inter-router link between R1 and R2 and the upstream connections to the ISP. These links provide basic Layer 3 connectivity and enable initial verification of router-to-router communication.

Interface descriptions are configured on all router and switch interfaces to clearly identify uplinks, downlinks, and end-host connections. This improves topology readability and simplifies verification during configuration and diagnostic steps.

<br>

## **3.2 Topology Diagram**


![](images/Pasted%20image%2020251220214739.png)

<br>

## **3.3 Objectives**

1. Assign static IPv4 addresses to the Xubuntu Server and Xubuntu Admin workstation.
    
2. Configure hostnames on all routers and switches.
    
3. Configure IPv4 addressing on point-to-point links between R1, R2, the ISP router, and the simulated Internet host.
    
4. Apply interface descriptions to all routing and switching devices.
    
5. Verify basic connectivity and router-to-router communication using interface status checks and ICMP tests.

<br>

## **3.4 Static IPv4 Addressing for Xubuntu Hosts**

Static IPv4 configuration is applied to the Xubuntu Server and Xubuntu Admin workstation because these systems represent administrator-managed infrastructure hosts.

### **Xubuntu Server (VLAN 10)**

The Xubuntu Server uses a static IPv4 address because it represents an infrastructure device that must remain reachable at a consistent address throughout all configuration stages.

**Open configuration file:**

```plaintext
sudo nano /etc/netplan/01-network-manager-all.yaml
```


**Configuration:**

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.90.10.10/26]
      nameservers:
        addresses: [10.90.10.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.90.10.1
```
![](images/Pasted%20image%2020251220222015.png)

**Apply configuration:**

```plaintext
sudo netplan apply
```


**Verification**:

```plaintext
ip a
```
![](images/Pasted%20image%2020251220222037.png)

### **Results:**

The server uses a static IPv4 address in the Server VLAN and is prepared for further configuration stages.


<br>

### **Xubuntu Admin (VLAN 20)**

The Xubuntu Admin workstation also receives a static IPv4 address to ensure reliable access for administrative and monitoring tasks during all later configuration steps.

**Configuration**:

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.90.20.10/26]
      nameservers:
        addresses: [10.90.20.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.90.20.1
```
![](images/Pasted%20image%2020251220222610.png)

**Apply configuration:**

```plaintext
sudo netplan apply
```


**Verification**:

```plaintext
ip a
```
![](images/Pasted%20image%2020251220222649.png)

### **Results**:

The admin workstation is correctly configured with a static IPv4 address and ready for management access.

<br>

## **3.5 Device Hostname Configuration**

This subsection defines hostname configuration for all network devices in the advanced topology. Hostnames are selected in a simple and consistent form to provide clear overview and easy identification of devices during configuration, verification, and later expansion of the network.

All routers and switches are assigned descriptive hostnames that directly reflect their role in the topology. End hosts (Xubuntu Server, Xubuntu Admin, Office PC, Warehouse PC, and Internet Host) already use predefined names assigned during topology creation and therefore do not require additional hostname configuration at this step.

### **Device Hostname Overview**

| **Device Type** | **Hostname** |
| ----------- | -------- |
| Router      | R1       |
| Router      | R2       |
| Router      | ISP      |
| Switch      | SW1      |
| Switch      | SW2      |
| Switch      | SW3      |
| Switch      | SW4      |

### **Hostname Configuration Example (Router R1)**

The example below shows hostname configuration on router R1. All other routers and switches are configured the same way, using the same commands with only the hostname value changed.

**Configuration:**

```plaintext
enable
configure terminal
hostname R1
exit
write
```
![](images/Pasted%20image%2020251220222932.png)

### **Results:**

All routers and switches now use consistent and descriptive hostnames. Each device is clearly identifiable in the CLI, diagnostic output, and topology, providing a clean baseline for all subsequent configuration stages.

<br>

## **3.6 IPv4 Configuration on Router Interfaces**

This section assigns IPv4 addresses to all router point-to-point interfaces. Each link uses a dedicated /30 subnet to provide clear separation between networks and reliable connectivity between directly connected devices. These interfaces form the external and inter-router Layer 3 backbone of the topology and are required for basic connectivity verification.

Only router interfaces used for point-to-point connections are configured in this step. Interfaces connected to the internal network and the management VLAN (VLAN 99) are left unconfigured and will be configured in later chapters.

### **Router R1**

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
ip address 10.90.200.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/5
ip address 192.0.2.2 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251220223142.png)

### **Router R2**

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
ip address 10.90.200.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/4
ip address 192.0.2.6 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251220223207.png)

### **ISP Router**

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 192.0.2.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/4
ip address 192.0.2.5 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/0
ip address 100.64.10.1 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251220223235.png)

### **Internet Host (VPCS)**

In this topology, the external Internet host is represented by a VPCS node. A static IPv4 address is assigned to simulate a reachable external endpoint for connectivity testing.

**Configuration:**

```plaintext
ip 100.64.10.2 255.255.255.252 100.64.10.1
```
![](images/Pasted%20image%2020251220223327.png)


This configuration assigns a fixed IPv4 address and default gateway to the simulated Internet host, allowing verification of connectivity between the internal routers and the external network.


### **Results:**

All router point-to-point interfaces and the simulated Internet host now use their assigned IPv4 addresses. Connectivity between R1, R2, the ISP router, and the Internet host is established and ready for verification in the next section.

<br>

## **3.7 Interface Description Configuration**

This part defines interface descriptions on routers and switches to improve link identification and topology clarity.

Descriptions are configured on all router and switch interfaces connected to other devices or end hosts. ISP router interfaces are not configured, as the ISP is treated as an external provider.

### **Port Description Mapping**

| **Device** | **Interface**   | **Description**       |
| ---------- | --------------- | --------------------- |
| R1         | Gi0/5           | Uplink to ISP         |
| R1         | Gi0/0           | Link to R2            |
| R1         | Gi0/1           | Downlink to SW1       |
| R2         | Gi0/4           | Uplink to ISP         |
| R2         | Gi0/0           | Link to R1            |
| R2         | Gi0/2           | Downlink to SW2       |
| SW1        | Gi0/1           | Uplink to R1          |
| SW1        | Gi0/2 and Gi0/0 | Redundant link to SW2 |
| SW1        | Gi0/3           | Downlink to SW3       |
| SW2        | Gi0/2           | Uplink to R2          |
| SW2        | Gi0/1 and Gi0/0 | Redundant link to SW1 |
| SW2        | Gi0/3           | Downlink to SW4       |
| SW3        | Gi0/0           | Link to SW4           |
| SW3        | Gi0/1           | Xubuntu Server        |
| SW3        | Gi0/2           | Xubuntu Admin         |
| SW3        | Gi0/3           | UpLink to SW1         |
| SW4        | Gi0/1           | Office PC             |
| SW4        | Gi0/2           | Warehouse PC          |
| SW4        | Gi0/0           | Link to SW3           |
| SW4        | Gi0/3           | Uplink to SW2         |


### **Configuration Example – Router R2**

The following example demonstrates interface description configuration on router R2. The same approach is used on all other routers and switches, with interface identifiers and description text adjusted according to the table above.

**Configuration**:

```plaintext
enable
configure terminal
interface Gi0/4
description Uplink to ISP
exit
interface Gi0/0
description Link to R1
exit
interface GigabitEthernet0/2
description Downlink to SW2
end
write
```
![](images/Pasted%20image%2020251220223408.png)

**verification:**

```
show running-config
```
![](images/Pasted%20image%2020251220223702.png)

### **Configuration Example – Switch SW3**

**Configuration**:

```plaintext
enable
configure terminal
interface Gi0/0
description Link to SW4
exit
interface Gi0/1
description Xubuntu Server
exit
interface Gi0/2
description Xubuntu Admin
exit
interface Gi0/3 
description UpLink to SW1
end
write
```
![](images/Pasted%20image%2020251220223959.png)

### **Results**:

All router and switch interfaces now include descriptive labels that reflect their connected devices or links. This improves topology clarity and provides a consistent baseline for all subsequent configuration chapters.


<br>

## **3.8 Connectivity Verification**

This section verifies that basic Layer 3 connectivity between core devices is working correctly. The checks confirm that router interfaces are up, IP addressing is applied as intended, and point-to-point links between routers and the ISP are reachable before continuing with VLAN and Layer 2 configuration.

### **Router Interface Status**

Interface status is checked to confirm that the configured router interfaces are operational and in the correct state.

**Example: Router R1**

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020251220224733.png)

The output confirms that point-to-point interfaces on R1 are up and have the expected IPv4 addresses assigned.

### **Ping Tests**

Ping tests are used to verify basic connectivity between directly connected devices.

**R1 Connectivity:**

- R1 → R2
    
- R1 → ISP
    

```plaintext
ping 10.90.200.2
ping 192.0.2.1
```
![](images/Pasted%20image%2020251220224841.png)

**R2 Connectivity:**

- R2 → R1
    
- R2 → ISP
    

```plaintext
ping 10.90.200.1
ping 192.0.2.5
```
![](images/Pasted%20image%2020251220224910.png)

**ISP Connectivity:**

- ISP → Internet Host
    

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251220224934.png)

Each successful ping confirms correct IPv4 configuration and reachability on the corresponding point-to-point link.

### **Neighbor Verification**

Neighbor visibility is verified to confirm correct physical cabling and Layer 2 connectivity between the core routers.

**Example: R2**

```plaintext
show cdp neighbors
```
![](images/Pasted%20image%2020251220224956.png)

The output confirms that router detects its directly connected neighbor on the expected interface.


### **Results**:

Basic connectivity and neighbor visibility between R1, R2, the ISP router, and the simulated Internet host are confirmed. All point-to-point links operate as expected, providing a stable baseline for VLAN and Layer 2 configuration in the next chapter.


<br>

## **3.9 Conclusion**

This chapter defines a stable baseline configuration for the advanced network topology. Static IPv4 addressing is configured on infrastructure hosts, hostnames are applied to all network devices, and IPv4 addressing is assigned to all point-to-point routing links. Interface descriptions are added to improve link identification, and basic connectivity and neighbor visibility are verified using standard diagnostic commands.

With Layer 3 connectivity confirmed and all core devices reachable, the network is ready for Layer 2 configuration. The next chapter focuses on VLAN implementation and switch-level settings, building on this baseline to introduce network segmentation and access-layer design.

<br>


---

<br>


Next chapter: 
















