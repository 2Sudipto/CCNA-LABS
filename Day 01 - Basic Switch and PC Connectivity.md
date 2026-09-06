# Day 01 - Basic Switch and PC Connectivity

## Overview

This lab was completed as part of my CCNA studies using Cisco Packet Tracer.

The purpose of this lab is to build a basic switch configuration from the ground up, apply IP addressing to both PCs and switches, and verify connectivity across the network using show commands and ping. This lab focuses on the foundational device access and management skills that every larger network build depends on.

---

## Network Topology

<p align="center">
  <img width="550" height="318" alt="image" src="https://github.com/user-attachments/assets/184e9a65-f272-4544-a375-c568e203f016" />
</p>

<p align="center">
  Two switches (S1 and S2) connected together, each supporting one PC. Both switches are configured with a management IP on VLAN 1, and both PCs are configured with static IPs on the same subnet.
</p>

---

## IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| S1 | VLAN 1 | 192.168.1.253 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.254 | 255.255.255.0 |
| PC1 | NIC | 192.168.1.1 | 255.255.255.0 |
| PC2 | NIC | 192.168.1.2 | 255.255.255.0 |

---

## Configuration Steps

### Configure Hostname, Passwords, and Banner on S1
```
Switch>enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#banner motd #Authorized access only. Violators will be prosecuted to the full extent of the law.#
```

Verifying that both passwords were actually applied is straightforward: exiting back to user mode and re-entering privileged EXEC mode should prompt for the enable secret, and disconnecting and reconnecting to the console line should prompt for the console password. Both can also be confirmed by checking `show running-config`.

### Save the Configuration
```
S1#copy running-config startup-config
```

### Repeat the Same Steps on S2
Same hostname, password, and banner configuration applied to S2, using `hostname S2` in place of S1.

---

### Configure the PCs
On PC1 and PC2, open the Desktop tab, go to IP Configuration, and enter the static IP and subnet mask from the addressing table above.

<p align="center">
 <img width="1672" height="540" alt="image" src="https://github.com/user-attachments/assets/63d1e9a8-076c-429e-9b37-92fd7f2129f0" />
 <img width="1788" height="448" alt="image" src="https://github.com/user-attachments/assets/c017b9d7-8c87-46bb-b78d-0985dd4c496d" />
</p>

### Test Connectivity to the Switches
From PC1's Command Prompt:
```
PC> ping 192.168.1.253
```

At this stage, the switches don't yet have an IP address configured, so this ping is expected to fail until the management interface is set up in the next part.
<img width="1632" height="504" alt="image" src="https://github.com/user-attachments/assets/ff44e7c5-84aa-4a8d-a578-1d8400db8151" />

---

### Configure the Switch Management Interface

Even though switches forward traffic based on MAC addresses and don't need an IP address to function as a plug-and-play Layer 2 device, an IP address is still required for remote management purposes, things like SSH access, monitoring, and troubleshooting. Without one, the only way to reach the switch's CLI is a direct console connection.

```
S1#configure terminal
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.253 255.255.255.0
S1(config-if)#no shutdown
```

The `no shutdown` command is necessary here because interfaces, including virtual ones like an SVI, come up in an administratively down state by default. Even with a valid IP address assigned, the interface won't pass traffic until it's explicitly brought up.

Repeat the same configuration on S2 using its address from the addressing table.

### Verify the IP Address Configuration
```
S1#show ip interface brief
S1#show running-config
```

<p align="center">
  <img width="791" height="471" alt="image" src="https://github.com/user-attachments/assets/e67724d5-4a67-4cec-a6ab-b78a900de305" />
  <img width="933" height="426" alt="image" src="https://github.com/user-attachments/assets/ee84aac1-c0d1-47d1-959b-991deaf021d2" />
</p>

### Save the Configuration Again
```
S1#copy running-config startup-config
```

### Verify Network Connectivity
From PC1's Command Prompt:
```
PC> ping 192.168.1.2
PC> ping 192.168.1.253
PC> ping 192.168.1.254
```

All pings should succeed. A first attempt occasionally showing partial success (like 80%) is normal and typically resolves to 100% on a second attempt, related to ARP resolution happening on the first packet.
<img width="856" height="514" alt="image" src="https://github.com/user-attachments/assets/b80e9dee-13b2-4b56-a89e-5999db2a6c36" />

---

## Key Concepts Demonstrated

- Basic Switch Configuration
- Console and Privileged EXEC Password Security
- MOTD Banners
- Switch Management via SVI (VLAN 1)
- Saving Configuration to NVRAM
- Static IP Addressing on End Devices
- Connectivity Verification with Ping

---

## What I Learned

This lab reinforced that a switch doesn't need an IP address to forward traffic, since switching decisions are based entirely on MAC addresses, but it does need one for remote management and troubleshooting. It also clarified why interfaces stay administratively down by default until a `no shutdown` is issued, even when an IP address is already assigned, which is an easy step to forget when a device isn't behaving as expected.

---

## Skills Practiced

- Basic Device Hardening (Passwords, Banner)
- Switch Management Interface Configuration
- Static IP Addressing
- Configuration Verification (`show ip interface brief`, `show running-config`)
- Saving Configuration to NVRAM
- Cisco Packet Tracer
