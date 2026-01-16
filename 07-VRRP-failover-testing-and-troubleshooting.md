
<br>

## **7.1 Introduction**

This chapter focuses on testing VRRP gateway redundancy in a simple and practical way. The network is already configured with VLANs, inter-VLAN routing, dynamic internal routing, Internet access, and core network services.

Before any failure is tested, the chapter first confirms that VRRP operates correctly during normal conditions. The roles of the core routers are verified to show that one router actively provides the default gateway while the second router remains ready to take over. Selected connectivity tests confirm that traffic flows correctly through the VRRP virtual gateway.

During the implementation of this chapter, a single NAT/PAT-related issue is identified while configuring Internet connectivity. The issue arises during normal network configuration and is not intentionally simulated. A dedicated troubleshooting section is included as part of this chapter to document the analysis and resolution of this problem.

In real networks, gateway redundancy is critical for availability. If one router fails, another router must immediately take over the gateway role so that users and services remain online. This chapter demonstrates that behavior by simulating a router failure and observing how the backup router automatically takes over the gateway role while the network remains operational.

<br>

## **7.2 Topology Diagram**

![](images/Pasted%20image%2020251228195632.png)


<br>

## **7.3 Objectives**

1. Verify that VRRP operates correctly during normal network operation.
    
2. Confirm that the primary router actively provides the default gateway for all VLANs.
    
3. Verify that the backup router remains ready to take over the gateway role.
    
4. Validate inter-VLAN and external connectivity through the VRRP virtual gateway.
    
5. Prepare a stable baseline for controlled VRRP failover testing.
    
6. Identify and resolve a real NAT/PAT connectivity issue that appears during network configuration.


<br>


## **7.4 VRRP Baseline Monitoring**

This section verifies correct VRRP operation under normal conditions before any failure is introduced. The goal is to confirm that the primary and backup router roles are established correctly and that the virtual gateway is active and stable.

### **VRRP Status Verification on Core Routers**

VRRP status is verified on both core routers to confirm active and standby roles, group IDs, priorities, and the virtual IP address. This establishes a clear and verified baseline for later failover testing.

**Router R1 (Primary Gateway)**

```prikaz
show vrrp brief
```
![](images/Pasted%20image%2020251228163608.png)


```prikaz
show vrrp
```
![](images/Pasted%20image%2020251228163653.png)


Result:

- Router R1 operates as the active VRRP gateway.
    
- The virtual IP address is owned by R1.
    
- VRRP priority on R1 is higher than on R2.
    

**Router R2 (Backup Gateway)**

```prikaz
show vrrp brief
```
![](images/Pasted%20image%2020251228163727.png)


```prikaz
show vrrp
```
![](images/Pasted%20image%2020251228163810.png)

**Result:**

- Router R2 operates in backup state.
    
- The virtual IP address is not owned by R2.
    
- Router R2 remains ready to take over the gateway role if required.
    

### **Wireshark Verification**

VRRP operation is observed using the native Wireshark capture feature in GNS3. Packet capture is started on the link between **Router R1 and Switch SW1**, which carries VLAN traffic toward the default gateway. This link represents the Layer 2 segment where the VRRP gateway operates for client VLANs.

**Link capture between R1 and SW1:**

![](images/Pasted%20image%2020251229183520.png)

```prikaz
vrrp
```
![](images/Pasted%20image%2020251228164401.png)

### **Results**

Wireshark capture confirms correct VRRP operation under normal conditions. VRRP advertisements are observed on the router-to-switch link for multiple VLANs, demonstrating that the virtual gateway is active and consistently advertised by the routers. The presence of regular VRRP announcement packets confirms that router roles are established correctly and that the VRRP control plane operates as expected before any failover testing is performed.

### **Summary**  

VRRP operates correctly under normal conditions. Router R1 provides the active default gateway using the VRRP virtual IP address, while Router R2 remains in backup state and is ready to take over if the primary router becomes unavailable. The verified baseline confirms that the network is prepared for controlled VRRP failover testing.

<br>

## **7.5 VRRP Baseline Traffic Verification**

This next section verifies that user traffic is correctly forwarded through the VRRP virtual gateway under normal operating conditions. 

The goal is to confirm that client-to-gateway, inter-VLAN, and external connectivity function as expected while Router R1 operates as the active VRRP master.

### **Test Scenario Selection**

Traffic verification is performed using representative end devices from different VLANs. The selected tests focus on typical communication paths used in real network operation and avoid unnecessary duplication of previously verified functionality.

The following endpoints are used:

- Office Client (VLAN 30)
    
- Xubuntu Server (VLAN 10)
    
- Internet Host (external network)
    

### **Client-to-Gateway Verification**

The Office client in VLAN 30 verifies reachability of the default gateway using the VRRP virtual IP address.

```prikaz
ping 10.90.30.1
```
![](images/Pasted%20image%2020251228170200.png)

This test confirms that the VRRP gateway responds correctly and that Router R1 actively provides gateway services for client VLANs.

### Inter-VLAN Connectivity Verification

The Office client verifies connectivity to the server VLAN, confirming correct inter-VLAN routing through the VRRP gateway.

```prikaz
ping 10.90.10.10
```
![](images/Pasted%20image%2020251228170211.png)

This test confirms that traffic from VLAN 30 is routed through the VRRP gateway toward VLAN 10 without interruption.

### **External Connectivity Verification**

The Office client verifies external connectivity by sending traffic toward the simulated Internet host. This confirms that traffic continues through the VRRP gateway and upstream routing and NAT infrastructure.

```prikaz
ping 100.64.10.1
```
![](images/Pasted%20image%2020251228170222.png)
### Results

All traffic verification tests complete successfully. The Office client reaches the VRRP virtual gateway, communicates with internal VLANs, and accesses the external network without packet loss. These results confirm that Router R1 correctly operates as the active VRRP master and that normal network traffic flows as expected prior to any failover testing.


<br>

## **7.6 VRRP Failover Test - Primary Router Outage**

This section tests VRRP failover behavior during a simulated primary router outage. Router R1 is intentionally powered off to simulate an unexpected router failure. The test observes VRRP role transition to Router R2 and verifies that network traffic continues to flow through the VRRP virtual gateway without manual intervention.

### **Simulate Primary Router Failure**

Router R1 is powered off in GNS3 to trigger VRRP failover.

Action:

- Stop Router R1 node (Power off)
    

### **Network Topology After R1 Shutdown**

![](images/Pasted%20image%2020251229183922.png)



### **Verify VRRP Role Transition**

After Router R1 is offline, Router R2 transitions to the active VRRP state and takes over the virtual gateway.

**Router R2 verification**

```prikaz
show vrrp brief
```
![](images/Pasted%20image%2020251228171405.png)

```prikaz
show vrrp
```
![](images/Pasted%20image%2020251228171436.png)

### **VRRP Monitoring During Failover**

VRRP behavior during the role transition is observed using Wireshark capture on the link between **Router R2 and Switch SW2**, which represents the active gateway path after failover.

```prikaz
vrrp
```
![](images/Pasted%20image%2020251228171614.png)


Wireshark observation:

- VRRP advertisements from Router R1 are no longer present.
    
- Router R2 sends VRRP advertisements as the active gateway.
    
- VRRP announcements continue at regular intervals after the transition.
    

### **Traffic Verification During Failover**

Traffic tests are repeated to confirm that client connectivity is maintained during the outage.

Office Client (VLAN 30):

```prikaz
ping 10.90.30.1
```
![](images/Pasted%20image%2020251228171725.png)

```prikaz
ping 10.90.20.10
```
![](images/Pasted%20image%2020251228171736.png)

```prikaz
ping 100.64.10.1
```
![](images/Pasted%20image%2020251228171745.png)

### Results

VRRP failover operates as designed. When Router R1 is powered off, Router R2 automatically takes over the VRRP virtual gateway role. VRRP monitoring confirms the role transition, and client traffic continues to reach the default gateway, the administrative VLAN, and the external network without manual changes.


<br>

## **7.7 NAT/PAT Troubleshooting – Internal VLAN Traffic Does Not Reach the Internet**


During the implementation of NAT/PAT in this project, an unexpected connectivity issue was found. After completing the configuration, internal VLANs were not able to reach the Internet host. This issue occurred during real project work and was identified and resolved through systematic troubleshooting.

### **1) Issue**

Hosts in all internal VLANs (VLAN 10, 20, 30, and 40) cannot reach the Internet host. Pings from routers to the Internet succeed, but pings from internal hosts fail.

**Ping Tests – Internal VLANs**  
    Connectivity to the Internet host is tested from each internal VLAN.
    

**VLAN 10 – Office**  
Source: PC in VLAN 10  
Destination: Internet Host (100.64.10.2)

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228195946.png)

**Result:** Ping fails.

**VLAN 20 – Admin**  
Source: PC in VLAN 20  
Destination: Internet Host (100.64.10.2)

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228200122.png)

**Result:** Ping fails.

**VLAN 30 – Server**  
Source: PC in VLAN 30  
Destination: Internet Host (100.64.10.2)

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228200158.png)

**Result:** Ping fails.

**VLAN 40 – Warehouse**  
Source: PC in VLAN 40  
Destination: Internet Host (100.64.10.2)

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228200031.png)

Result: Ping fails.

These tests confirm that the issue affects all internal VLANs.


<br>


### 2) **Diagnostics**

The troubleshooting process starts by validating the basic assumptions of external connectivity. Each segment of the traffic path is checked step by step to determine where communication fails.


##### **Internet Host**

```plaintext
ip address
```
![](images/Pasted%20image%2020251228200219.png)

The Internet host has a valid IP address and default gateway.

Next, connectivity and routing are verified on the ISP router to confirm that the upstream network operates correctly.

##### **ISP Router**

```plaintext
show ip interface brief
show ip route
```
![](images/Pasted%20image%2020251228200247.png)

The ISP router shows correct interface status and routing information.



<br>

### **3) Root Cause**  

The NAT configuration is incorrect. The `ip nat inside` command is applied only to the physical trunk interface instead of the VLAN subinterfaces. As a result, traffic from internal VLANs is not translated and cannot reach external networks.

The NAT configuration on Router R2 is reviewed to identify how NAT inside and outside are applied.

```plaintext
show running-config | section nat
```
![](images/Pasted%20image%2020251228200616.png)

The output shows that `ip nat inside` is configured only on the physical trunk interface (Gi0/1 / Gi0/2), while the VLAN subinterfaces do not have NAT inside configured.

In a router-on-a-stick design, Layer 3 traffic is processed on subinterfaces, not on the parent physical interface.

<br>

### 4) **Fix**  

Once the root cause is identified, the NAT/PAT configuration is adjusted to align with the router-on-a-stick design used in the project.


##### **Router R1**

```plaintext
configure terminal
interface GigabitEthernet0/1
no ip nat inside
exit
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
end
write
```
![](images/Pasted%20image%2020251228200937.png)

##### **Router R2**

```plaintext
configure terminal
interface GigabitEthernet0/2
no ip nat inside
exit
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
end
write
```
![](images/Pasted%20image%2020251228201018.png)

**Results:**

The problem is fixed by moving `ip nat inside` from the physical interfaces to the VLAN subinterfaces on R1 and R2.

<br>

## **5) Verification**  

After applying the configuration changes, connectivity and NAT behavior are validated to confirm that the issue is fully resolved.


Internet connectivity is tested again from all internal VLANs.

**VLAN 30 – Office**

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228201040.png)

**Result:** Ping is successful.

**VLAN 20 – Admin**

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228201105.png)

**Result:** Ping is successful.

**VLAN 10 – Server**

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228201150.png)

**Result:** Ping is successful.

**VLAN 40 – Warehouse**

```plaintext
ping 100.64.10.2
```
![](images/Pasted%20image%2020251228201208.png)

**Result:** Ping is successful.

NAT operation is verified on Router R1 (Master) to confirm that translations are created and traffic is processed correctly.

```plaintext
show ip nat translations
show ip nat statistics
```
![](images/Pasted%20image%2020251228201500.png)

NAT translations are created correctly and hit counters increase as traffic passes through the router.

### **Summary**  

The issue was caused by an incorrect NAT/PAT configuration in a router-on-a-stick design. Applying `ip nat inside` only to the physical trunk interface prevented address translation for VLAN traffic. After moving the NAT inside configuration to the VLAN subinterfaces, Internet connectivity was restored for all internal VLANs.

<br>

## **7.7 Conclusion**

This chapter confirms correct VRRP gateway redundancy during both normal operation and a simulated primary router failure. VRRP roles, virtual gateway availability, and traffic flow are validated to establish a stable baseline for the network.

When the primary router is powered off, the backup router successfully assumes the active gateway role and maintains uninterrupted client connectivity without manual intervention. Monitoring and traffic verification confirm consistent network behavior throughout the failover process.

A real NAT/PAT issue identified during network implementation is also addressed. The problem is analyzed and resolved using clear troubleshooting steps, ensuring reliable connectivity and stable network operation.

The next and final chapter closes the project and gives a clear overview of the entire network. It briefly reviews all chapters and summarizes the main configurations and results.

<br>

---


<br>


**Final chapter:** [Conclusion and Summary](08-conclusion-and-summary.md)


























































