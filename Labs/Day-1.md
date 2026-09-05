# Day 01 - VLANs and TRUNK

## Overview

This lab was completed as part of my CCNA studies using Cisco Packet Tracer.

The purpose of this lab is to learn how to create VLANs, assign ports to different VLANs, and trunk two switches so that devices in the same VLAN can communicate with each other across switches.

---

## Network Topology

<p align="center">
  <img width="1094" height="386" alt="image" src="https://github.com/user-attachments/assets/969e550f-3cfc-4a1d-a06d-1a1683694f06" />
</p>

<p align="center">
 Two Cisco 2960 switches (SW1 and SW2) connected by a trunk link, each hosting a Sales PC on VLAN 100 (192.168.10.0/24) and an IT PC on VLAN 200 (192.168.20.0/24). Same-VLAN devices across switches communicate successfully through the trunk, while devices in different VLANs remain isolated even when on the same switch.
</p>

---

## IP Addressing

| VLAN/Segment | Network |
|---|---|
| VLAN 100 | 192.168.10.0/24 |
| VLAN 200 | 192.168.20.0/24 |

---

## Configuration Steps
1. Create VLAN 100 and VLAN 200 on both switches and verify
2. Assign each PC's port to the correct VLAN
3. Configure the trunk link between the two switches
4. Assign static IPs to all 4 PCs
5. Verify VLANs and trunk with show commands
6. Ping same-VLAN devices across switches — confirm success
7. Ping different-VLAN devices — confirm failure

### [Step/Feature 1]
```
SW1>en
SW1#config t
Enter configuration commands, one per line.  End with CNTL/Z.
SW1(config)#vlan 100
SW1(config-vlan)#name SALES
SW1(config-vlan)#vlan 200
SW1(config-vlan)#name IT
SW1(config-vlan)#
!
!
!
SW2>en
SW2#config t
Enter configuration commands, one per line.  End with CNTL/Z.
SW2(config)#vlan 100
SW2(config-vlan)#name SALES
SW2(config-vlan)#vlan 200 
SW2(config-vlan)#name IT
```
<p align="center">
<img width="612" height="272" alt="image" src="https://github.com/user-attachments/assets/35d3992c-01f5-4330-84f5-79f5f98a492c" />
<img width="614" height="228" alt="image" src="https://github.com/user-attachments/assets/4777229d-31f3-4da7-9b8e-20ca8dd17794" />
</p>

### [Step/Feature 2]
```
SW1(config)#int f0/1
SW1(config-if)#no shut
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 200
SW1(config-if)#exit
SW1(config)#int f0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 100
SW1(config-if)#no shut
!
SW2(config)#int f0/1
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 100
SW2(config-if)#no shut
SW2(config-if)#exit
SW2(config)#int f0/2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 200
SW2(config-if)#no shut
SW2(config-if)#exit
```
<p align="center">
<img width="1184" height="576" alt="image" src="https://github.com/user-attachments/assets/c7a0c9a8-1766-4285-9a97-4d588dcca59e" />
<img width="1378" height="736" alt="image" src="https://github.com/user-attachments/assets/3173255b-4935-4194-a39d-f385e9ba8aaf" />
</p>

### [Step/Feature 3]
Trunk the switches
<p align="center">
  <img width="1024" height="678" alt="image" src="https://github.com/user-attachments/assets/9d12be58-b7a9-4984-8a00-d476480e57d0" />

</p>

### [Step/Feature 4]
---
Assign IP addresses to the PCs
<p align="center">
<img width="770" height="336" alt="image" src="https://github.com/user-attachments/assets/cacb1992-622a-4715-a2e4-6609954773a4" />
<img width="1026" height="266" alt="image" src="https://github.com/user-attachments/assets/c6ef3b2a-573b-4a1d-9a9e-927340761f28" />
</p>
Do the same on the other two PCs

## Verification

<p align="center">
<img width="1088" height="890" alt="image" src="https://github.com/user-attachments/assets/59a1ef3f-3176-4495-9fd6-7f5869c08629" />
<img width="1028" height="930" alt="image" src="https://github.com/user-attachments/assets/5e414495-3d39-469d-b94c-feeadcddd785" />
<img width="1060" height="824" alt="image" src="https://github.com/user-attachments/assets/9c94732b-6c2e-4b1c-b58b-77601f64810d" />
</p>

---

## Key Concepts Demonstrated

- VLAN Creation and Naming
- Access Port to VLAN Assignment
- Trunk Port Configuration
- VLAN Isolation vs. Same-VLAN Communication Across Switches
- Static IP Addressing


---

## What I Learned


This lab helped me understand that VLANs create logical separation independent of physical switch placement — devices in the same VLAN can communicate across a trunk link even when they're on different switches, while devices in different VLANs stay isolated even on the same switch. I also learned that a trunk port doesn't create communication between different VLANs — it only allows multiple VLANs' traffic to travel across a single physical link, with each VLAN's isolation still enforced on the other end.

---

## Skills Practiced

- VLAN Configuration and Verification
- Access Port Assignment
- Trunk Port Configuration
- Static IP Address Assignment
- Network Verification (`show vlan brief`, `show interfaces trunk`, `ping`)
- Cisco Packet Tracer



