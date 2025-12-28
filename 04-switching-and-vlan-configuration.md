
<br>


# **4 - Switching and VLAN Configuration**

<br>

## **4.1 Introduction**

This chapter defines the Layer 2 switching configuration for the advanced redundant network design. VLANs are used to separate the network into logical segments and to control Layer 2 traffic in a clear and predictable way.

The switching layer is prepared for redundancy. Trunk links are configured on both sides of the network so that all VLANs remain available even if one link or routing device fails. This design supports traffic continuity and prepares the network for later gateway redundancy testing.

Access ports are assigned to end devices according to the VLAN plan. Trunk ports carry all required VLANs, including the management VLAN 99, across the entire switching infrastructure. Access ports are additionally protected using PortFast and BPDU Guard to ensure fast host connectivity and to prevent Layer 2 loops caused by incorrect device connections.

This chapter focuses only on Layer 2 configuration. Inter-VLAN routing and virtual gateway redundancy are configured in later chapters.

<br>

## **4.2 Topology Diagram**

![](images/Pasted%20image%2020251224135629.png)

<br>

## **4.3 VLAN Overview**

| **VLAN ID** | **VLAN Name** | **Description / Purpose**      | **Assigned Devices**         | **Switch Association** |
| ----------: | ------------- | ------------------------------ | ---------------------------- | ---------------------- |
|          10 | Server        | Internal server infrastructure | Xubuntu-Server               | SW3                    |
|          20 | Admin         | Administrative workstations    | Xubuntu-Admin                | SW3                    |
|          30 | Office        | Office user workstations       | Office PC                    | SW4                    |
|          40 | Warehouse     | Warehouse user workstations    | Warehouse PC                 | SW4                    |
|          99 | Management    | Network device management      | Switch management interfaces | SW1, SW2, SW3, SW4     |

All VLANs are created on every switch to ensure consistent Layer 2 forwarding across trunk links. VLAN 99 is present on all switches to support centralized management.

<br>

## **4.4 Objectives**


1. Create all required VLANs on every switch according to the network design.
    
2. Assign access ports to the correct VLANs based on connected end devices.
    
3. Configure trunk ports to carry all required VLANs across the switching layer.
    
4. Configure Rapid Spanning Tree Protocol (RSTP) to control Layer 2 topology and root bridge roles.
    
5. Configure link aggregation between distribution switches using LACP.
    
6. 6. Configure PortFast and BPDU Guard on access ports to protect the Layer 2 topology.
    
7. Verify VLAN, trunk, spanning tree, and EtherChannel configuration using diagnostic commands.

<br>

## **4.5 VLAN Creation on Switches**

This section creates all required VLANs on the switching layer according to the network design. The distribution switches (SW1 and SW2) provide the main Layer 2 redundancy in the network, while the access switches (SW3 and SW4) participate in this design and extend redundant paths through the access layer.

Each switch uses the same VLAN database to ensure consistent Layer 2 forwarding across trunk links and redundant paths.

All VLANs are created on every switch, even if no local end device is connected to a specific VLAN. This approach ensures that trunk links can forward traffic for all VLANs across the entire switching infrastructure.

#### **Switch SW1**

```plaintext
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Office
exit
vlan 40
name Warehouse
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251224135809.png)

#### **Switch SW2**

```plaintext
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Office
exit
vlan 40
name Warehouse
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251224135847.png)

#### **Switch SW3**

```plaintext
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Office
exit
vlan 40
name Warehouse
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251224135926.png)

#### **Switch SW4**

```plaintext
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Office
exit
vlan 40
name Warehouse
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251224135957.png)

### **Results**

All switches now have the full VLAN list required for the advanced redundant topology. The VLAN database is consistent across the switching layer and ready for access port and trunk configuration.


<br>

## **4.6 Assign Access Ports to VLANs**

This section assigns access ports to the correct VLANs based on the physical topology and VLAN planning. Each end device is connected to a dedicated access port, and the port is placed into the VLAN that matches the device role.

Access ports are configured only on the access layer switches. Distribution switches are not used for direct end-device connectivity.

#### **Switch SW3**

```plaintext
enable
configure terminal
interface gi0/1
switchport mode access
switchport access vlan 10
no shutdown
exit
interface gi0/2
switchport mode access
switchport access vlan 20
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140053.png)

#### **Switch** SW4

```plaintext
enable
configure terminal
interface gi0/1
switchport mode access
switchport access vlan 30
no shutdown
exit
interface gi0/2
switchport mode access
switchport access vlan 40
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140123.png)

### **Results**

All end devices are connected to access ports and assigned to the correct VLANs. Layer 2 segmentation is now enforced at the access layer, and the switching infrastructure is ready for trunk and spanning tree configuration.



<br>

## **4.7 Configure Trunk Ports**

This section configures trunk links between switches and toward the routers. Trunk ports carry tagged VLAN traffic across the switching layer and ensure that all required VLANs are available across both sides of the redundant topology.

All trunk links allow VLANs 10,20,30,40,99.

#### **Switch SW1**

Trunk ports on SW1:

- Gi0/0 (to SW2)
    
- Gi0/2 (to SW2)
    
- Gi0/3 (to SW3)
    
- Gi0/1 (to R1)
    

```plaintext
enable
configure terminal
interface range gi0/0,gi0/2,gi0/3,gi0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140158.png)

#### **Switch SW2**

Trunk ports on SW2:

- Gi0/0 (to SW1)
    
- Gi0/1 (to SW1)
    
- Gi0/3 (to SW4)
    
- Gi0/2 (to R2)
    

```plaintext
enable
configure terminal
interface range gi0/0,gi0/1,gi0/3,gi0/2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140225.png)

#### **Switch SW3**

Trunk ports on SW3:

- Gi0/3 (to SW1)
    
- Gi0/0 (to SW4)
    

```plaintext
enable
configure terminal
interface range gi0/3,gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140252.png)

#### **Switch SW4**

Trunk ports on SW4:

- Gi0/3 (to SW2)
    
- Gi0/0 (to SW3)
    

```plaintext
enable
configure terminal
interface range gi0/3,gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224140316.png)

### **Results**

All inter-switch links and router uplinks are configured as trunk ports and allow VLANs 10,20,30,40,99. The switching layer is now ready for spanning tree and EtherChannel configuration.



>**Notes:** IOSv-L2 switches in GNS3 require the manual command _switchport trunk encapsulation dot1q_ on every trunk interface because 802.1Q is not enabled by default.


<br>

## **4.8 RSTP Configuration**

This section configures Rapid Spanning Tree Protocol (RSTP) on the switching layer. RSTP is used to prevent Layer 2 loops and to control active forwarding paths in the redundant topology.

RSTP mode is enabled on all switches to keep the entire switching layer consistent. Root bridge roles are then explicitly defined on the distribution switches to keep spanning tree control within the distribution layer.

### **Enable RSTP on All Switches (Example: SW1)**

RSTP is configured on all switches in the topology. For demonstration purposes, the configuration below shows the setup on SW1 only. The same configuration is applied on SW2, SW3, and SW4.

````plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
end
write
````
![](images/Pasted%20image%2020251224140413.png)

### **Configure Root Bridge Roles**

The distribution switches are configured with explicit bridge priorities. SW1 operates as the primary root bridge, while SW2 operates as the secondary root bridge for all VLANs.

#### **Switch SW1 (Root Primary)**

```plaintext
enable
configure terminal
spanning-tree vlan 10,20,30,40,99 priority 4096
end
write
```
![](images/Pasted%20image%2020251224140555.png)

#### **Switch SW2 (Root Secondary)**

```plaintext
enable
configure terminal
spanning-tree vlan 10,20,30,40,99 priority 8192
end
write
```
![](images/Pasted%20image%2020251224140627.png)

### **Verification**

RSTP operation and root bridge placement are verified on the distribution switches.

#### **Switch SW1**

```plaintext
show spanning-tree summary
```
![](images/Pasted%20image%2020251224140831.png)

#### **Switch SW2**

````plaintext
show spanning-tree summary
````
![](images/Pasted%20image%2020251224140922.png)

### **Results**

RSTP is enabled on all switches using rapid-pvst mode. SW1 is elected as the primary root bridge and SW2 as the secondary root bridge for all VLANs. The Layer 2 topology is protected against loops and prepared for stable redundant forwarding paths.

<br>

## **4.9 LACP (EtherChannel) Configuration**

This section configures Link Aggregation Control Protocol (LACP) between the distribution switches. EtherChannel is used to bundle multiple physical links into a single logical link, increasing bandwidth and providing link-level redundancy.

LACP is configured only between SW1 and SW2, where parallel links exist. **The access layer is not involved in EtherChannel configuration.**

#### **EtherChannel Overview**

- Physical links between SW1 and SW2 are grouped into one logical Port-Channel
    
- All member interfaces operate as trunk ports
    
- Spanning Tree treats the Port-Channel as a single link
    

### **Configuration**:

#### **Switch SW1**

The following interfaces are added to EtherChannel using LACP in active mode.

```plaintext
enable
configure terminal
interface range gi0/0,gi0/2
channel-group 1 mode active
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224141426.png)

#### **Switch SW2**

The following interfaces are added to EtherChannel using LACP in active mode.

```plaintext
enable
configure terminal
interface range gi0/0,gi0/1
channel-group 1 mode active
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251224141852.png)

### **Verification**

EtherChannel operation and LACP status are verified on both distribution switches.

##### **Switch SW1**

```plaintext
show etherchannel summary
```
![](images/Pasted%20image%2020251224142453.png)

##### **Switch SW2**

 ```
show etherchannel summary
show interfaces port-channel 1
 ```
![](images/Pasted%20image%2020251224144237.png)

### **Results**

The physical links between SW1 and SW2 are successfully bundled into a single Port-Channel using LACP. The switching layer now provides increased bandwidth and stable link redundancy between the distribution switches.

<br>

### **Port-Channel Description**

The Port-Channel interface uses a description to clearly identify the logical uplink between distribution switches.

Configuration:

SW1: 
 ```
interface port-channel 1  
description LACP trunk to SW2
 ```
![](images/Pasted%20image%2020251224150757.png)

SW2:  
 ```
interface port-channel 1  
description LACP trunk to SW1
 ```
![](images/Pasted%20image%2020251224150735.png)

>**Note.:** The description is applied only on the Port-Channel interface, not on individual member ports.

<br>

## **4.10 PortFast and BPDU Guard Configuration**

This section configures PortFast and BPDU Guard on access ports. These features are used to protect the Layer 2 topology and to prevent simple spanning tree issues caused by end devices.

PortFast allows an access port to become active immediately when a device is connected, without waiting for spanning tree timers. BPDU Guard protects the network by disabling an access port if a BPDU is received, which helps prevent accidental connection of switches.

PortFast and BPDU Guard are applied only to access ports. Trunk ports are not affected.

For demonstration purposes, the configuration below shows the setup on SW3 only. The same configuration is applied to all access switches in the topology.

#### **Enable PortFast on Access Ports (Example: SW3)**

```plaintext
enable
configure terminal
interface range gi0/1,gi0/2
spanning-tree portfast
exit
end
write
```
![](images/Pasted%20image%2020251224143131.png)

#### **Enable BPDU Guard on Access Ports (Example: SW3)**

```plaintext
enable
configure terminal
interface range gi0/1,gi0/2
spanning-tree bpduguard enable
exit
end
write
```
![](images/Pasted%20image%2020251224143500.png)

#### **Verification**

```plaintext
show spanning-tree interface gi0/1 detail
show spanning-tree summary
```
![](images/Pasted%20image%2020251224143936.png)

### **Results**

PortFast and BPDU Guard are enabled on access ports. The switching layer is protected against accidental loops and unauthorized switches connected to access interfaces.

<br>

## **4.11 Verification**

This section verifies VLAN presence, trunk operation, and interface status on each switch. The verification focuses on core Layer 2 functionality using a minimal and consistent set of diagnostic commands.

#### Verification Commands (Example: SW1)

```plaintext
show vlan brief
show spanning-tree summary
```
![](images/Pasted%20image%2020251224144518.png)

 ```
show interfaces trunk
show interfaces status
 ```
![](images/Pasted%20image%2020251224150831.png)


These commands are executed on all switches in the topology. The example above shows verification on SW1 only.

### **Results**

All required VLANs are present and active on the switches. Trunk ports are operational and carry the expected VLANs. Interface status confirms that access and trunk ports are in the correct operational state. The Layer 2 switching configuration is verified and ready for use.


<br>

## **4.12 Conclusion**

VLANs are consistently defined across all switches, access ports are correctly assigned to end devices, and trunk links ensure VLAN availability **across the switching layer**. Redundant paths are controlled using RSTP, with root bridge roles kept in the distribution layer to maintain a stable and predictable topology.

Link aggregation between the distribution switches improves **reliability** and simplifies Layer 2 forwarding **by grouping parallel links into a single logical connection**. Access ports are additionally protected using PortFast and BPDU Guard to reduce the risk of loops caused by incorrect device connections. The switching infrastructure is now verified and prepared for Layer 3 configuration in the following chapters.

<br>


---


<br>

**Next chapter:**