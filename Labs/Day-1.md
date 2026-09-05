# Day 01 - [VLANs and TRUNK]

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
---

## Verification

[What was checked to confirm it worked — show commands, ping results. Screenshots go here if you have them.]

<p align="center">
  <img src="https://github.com/<your-username>/<your-repo>/blob/main/screenshots/day-XX-verification.png" alt="Verification" width="800">
</p>

---

## Key Concepts Demonstrated

- [Concept 1]
- [Concept 2]
- [Concept 3]

---

## What I Learned

[2-4 sentences, genuine — what actually clicked, what was tricky, any real troubleshooting worth mentioning. This is the section that reads as authentic rather than a checklist.]

---

## Skills Practiced

- [Skill 1]
- [Skill 2]
- [Skill 3]
- Cisco Packet Tracer



