# Day 02 - Static Routing Between Two Sites (NY and NJ)

## Overview

This lab was completed as part of my CCNA studies using Cisco Packet Tracer.

The purpose of this lab is to connect two sites, NY and NJ, over a serial WAN link and use static routes so every LAN can reach every other LAN. Each site has two routed LANs behind a router and a switch per LAN. On top of the routing, the lab also covers basic hardening: one shared privilege 15 account for console and SSHv2 access, an enable secret on every device, and port security on the access ports.

---

## Network Topology

<p align="center">
  <img width="1661" height="727" alt="image" src="https://github.com/user-attachments/assets/494db151-1197-45a8-8ad5-5c73d00149a9" />
</p>

<p align="center">
  NY_Router and NJ_Router are connected by a 1.1.1.0/30 serial link. Each router has two LAN-facing interfaces, and each LAN has one switch with two PCs. The switches are named after the department they serve: Sales, Production, HR, and Marketing. NY_Router is the DCE side of the serial link.
</p>

---

## IP Addressing

### Routers

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| NY_Router | Gi0/0/0 | 192.168.1.1 | 255.255.255.0 |
| NY_Router | Gi0/0/1 | 192.168.2.1 | 255.255.255.0 |
| NY_Router | S0/1/0 (DCE) | 1.1.1.1 | 255.255.255.252 |
| NJ_Router | Gi0/0/0 | 192.168.3.1 | 255.255.255.0 |
| NJ_Router | Gi0/0/1 | 192.168.4.1 | 255.255.255.0 |
| NJ_Router | S0/1/0 | 1.1.1.2 | 255.255.255.252 |

### Switches and PCs

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| Sales | VLAN 1 | 192.168.1.254 | 255.255.255.0 | 192.168.1.1 |
| Production | VLAN 1 | 192.168.2.254 | 255.255.255.0 | 192.168.2.1 |
| HR | VLAN 1 | 192.168.3.254 | 255.255.255.0 | 192.168.3.1 |
| Marketing | VLAN 1 | 192.168.4.254 | 255.255.255.0 | 192.168.4.1 |
| PC0 | NIC | 192.168.1.4 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.5 | 255.255.255.0 | 192.168.1.1 |
| PC0(1) | NIC | 192.168.2.4 | 255.255.255.0 | 192.168.2.1 |
| PC1(1) | NIC | 192.168.2.5 | 255.255.255.0 | 192.168.2.1 |
| PC0(2) | NIC | 192.168.3.4 | 255.255.255.0 | 192.168.3.1 |
| PC1(2) | NIC | 192.168.3.5 | 255.255.255.0 | 192.168.3.1 |
| PC1(3) | NIC | 192.168.4.4 | 255.255.255.0 | 192.168.4.1 |
| PC0(3) | NIC | 192.168.4.5 | 255.255.255.0 | 192.168.4.1 |

### Device Access

All routers and switches use the same credentials.

| Setting | Value |
|---|---|
| Local user (console and SSH) | sshadmin |
| Password | sshpw |
| Privilege level | 15 |
| Enable secret | cisco |
| SSH version | 2 |
| Domain name | lab.local |

> Lab credentials only. Never reuse real passwords in a public repo.

---

## Configuration Steps

### Configure Hostname, Enable Secret, and Banner on NY_Router
```
Router>enable
Router#configure terminal
Router(config)#hostname NY_Router
NY_Router(config)#no ip domain-lookup
NY_Router(config)#enable secret cisco
NY_Router(config)#service password-encryption
NY_Router(config)#banner motd #Authorized access only. Violators will be prosecuted to the full extent of the law.#
```

`no ip domain-lookup` stops the router from trying to resolve mistyped commands as hostnames, which saves a lot of waiting around. `service password-encryption` makes sure any plain text line passwords in the config get obscured.

### Configure the Router Interfaces
```
NY_Router(config)#interface gigabitEthernet 0/0/0
NY_Router(config-if)#ip address 192.168.1.1 255.255.255.0
NY_Router(config-if)#no shutdown
NY_Router(config-if)#exit
NY_Router(config)#interface gigabitEthernet 0/0/1
NY_Router(config-if)#ip address 192.168.2.1 255.255.255.0
NY_Router(config-if)#no shutdown
NY_Router(config-if)#exit
NY_Router(config)#interface serial 0/1/0
NY_Router(config-if)#ip address 1.1.1.1 255.255.255.252
NY_Router(config-if)#clock rate 64000
NY_Router(config-if)#no shutdown
```

NY_Router is the DCE side of the serial link, so it is the one that needs `clock rate`. Without it the link never comes up. Router interfaces are also administratively down by default, so every one of them needs a `no shutdown`.

### Repeat on NJ_Router
Same hostname, enable secret, and banner steps with `hostname NJ_Router`. Use the NJ_Router addresses from the table above. Do not add `clock rate` on NJ_Router, since it is the DTE side.

### Configure the Static Routes
On NY_Router, point to the NJ LANs through NJ_Router's serial address:
```
NY_Router(config)#ip route 192.168.3.0 255.255.255.0 1.1.1.2
NY_Router(config)#ip route 192.168.4.0 255.255.255.0 1.1.1.2
```

On NJ_Router, point to the NY LANs through NY_Router's serial address:
```
NJ_Router(config)#ip route 192.168.1.0 255.255.255.0 1.1.1.1
NJ_Router(config)#ip route 192.168.2.0 255.255.255.0 1.1.1.1
```

Routes have to exist in both directions. A ping needs a path there and a path back, so if only one router has routes, the echo request arrives but the reply gets dropped.

---

### Configure the Switch Management Interface and Gateway
```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Sales
Sales(config)#enable secret cisco
Sales(config)#interface vlan 1
Sales(config-if)#ip address 192.168.1.254 255.255.255.0
Sales(config-if)#no shutdown
Sales(config-if)#exit
Sales(config)#ip default-gateway 192.168.1.1
```

The default gateway matters here. Without it the switch can only be reached from devices on its own subnet, so SSHing into Sales from the NJ side would fail even with perfect routing.

Repeat on Production, HR, and Marketing using their own hostnames and addresses from the table.

### Configure the PCs
On each PC, open the Desktop tab, go to IP Configuration, and enter the static IP, subnet mask, and default gateway from the addressing table.

<p align="center">
  <img width="800" alt="image" src="PASTE_PC_CONFIG_IMAGE_LINK" />
</p>

---

### Configure the Local User, Console Login, and SSHv2
These commands are identical on every router and switch, since all devices share the same account. Hostname has to be set first. The example below is on NY_Router.
```
NY_Router(config)#ip domain-name lab.local
NY_Router(config)#username sshadmin privilege 15 secret sshpw
NY_Router(config)#crypto key generate rsa modulus 2048
NY_Router(config)#ip ssh version 2
NY_Router(config)#line console 0
NY_Router(config-line)#login local
NY_Router(config-line)#exit
NY_Router(config)#line vty 0 15
NY_Router(config-line)#login local
NY_Router(config-line)#transport input ssh
NY_Router(config-line)#exit
```

The same local account is used for both the console and SSH, so there is only one username and password to manage. Privilege level 15 means that account lands straight in privileged EXEC mode after logging in, with no `enable` step. The enable secret is still set for anyone who ends up in user EXEC.

SSH needs a hostname and a domain name before it can generate the RSA keys, which is why those come first. `transport input ssh` blocks Telnet, so anything sent to the vty lines is encrypted. The user is created before `login local` is applied to the console so there is no risk of getting locked out.

### Configure Port Security on the Switches
Apply this to the PC-facing access ports only, not the uplink to the router.
```
Sales(config)#interface range fastEthernet 0/1 - 2
Sales(config-if-range)#switchport mode access
Sales(config-if-range)#switchport port-security
Sales(config-if-range)#switchport port-security maximum 1
Sales(config-if-range)#switchport port-security violation shutdown
Sales(config-if-range)#switchport port-security mac-address sticky
```

Sticky learns the MAC address of the first device that sends traffic on the port and saves it to the running config. With a max of 1 and violation set to shutdown, plugging a different device into that port puts it in err-disabled state.

### Save the Configuration on Every Device
```
NY_Router#copy running-config startup-config
```

---

## Verification

### Verify the Interfaces
On both routers:
```
NY_Router#show ip interface brief
NJ_Router#show ip interface brief
```

All three interfaces on each router should show `up` and `up`.

<p align="center">
  <img width="800" alt="image" src="PASTE_INTERFACE_BRIEF_IMAGE_LINK" />
</p>

### Verify the WAN Link
From NY_Router:
```
NY_Router#ping 1.1.1.2
```

This should succeed. If it fails, check `clock rate` on the DCE side and the interface status on both routers.

<p align="center">
  <img width="767" height="260" alt="image" src="https://github.com/user-attachments/assets/c4067d2e-5309-4756-9d0a-94c6ac302357" />
</p>

### Verify the Routing Tables
```
NY_Router#show ip route
NJ_Router#show ip route
```

The static routes should show up with an `S` code, alongside the `C` and `L` entries for the directly connected networks.

<p align="center">
  <img width="417" height="167" alt="image" src="https://github.com/user-attachments/assets/6cdf7090-2bd6-4fc3-a163-2ed3ae469a7c" />
  <img width="610" height="237" alt="image" src="https://github.com/user-attachments/assets/ba9e5fce-d83d-4fbc-b7a8-90e45f56a9a2" />
</p>

### Verify Connectivity Between LANs in the Same Site
From PC0 (Sales LAN) in NY:
```
PC> ping 192.168.2.4
```

From PC0(2) (HR LAN) in NJ:
```
PC> ping 192.168.4.4
```

Both go through the local router, so they confirm each router is routing between its own LANs.

### Verify Connectivity Between Sites
From PC0 in NY:
```
PC> ping 192.168.3.4
PC> ping 192.168.4.4
```

From PC0(2) in NJ:
```
PC> ping 192.168.1.4
PC> ping 192.168.2.3
```

All pings should succeed. A first attempt sometimes shows a dropped packet, which is normal and comes from ARP resolution on the first packet.

<p align="center">
  <img width="800" alt="image" src="PASTE_PING_SUCCESS_IMAGE_LINK" />
</p>

### Verify the Path Across the WAN
From PC0 in NY:
```
PC> tracert 192.168.3.4
```

The first hop should be 192.168.1.1 (NY_Router) and the second hop 1.1.1.2 (NJ_Router), which shows traffic crossing the serial link.

<p align="center">
  <img width="800" alt="image" src="PASTE_TRACERT_IMAGE_LINK" />
</p>

### Verify Port Security
Run this after the pings so the sticky MAC addresses have been learned.
```
Sales#show port-security
Sales#show port-security interface fastEthernet 0/1
```

<p align="center">
  <img width="800" alt="image" src="PASTE_PORT_SECURITY_IMAGE_LINK" />
</p>

### Verify SSH Access
From PC0(2) in NJ, SSH into the Sales switch in NY:
```
PC> ssh -l sshadmin 192.168.1.254
```

After entering the password, the prompt should land directly in privileged EXEC mode (`Sales#`) because the account is privilege 15. This also proves the switch default gateway and the static routes work for management traffic.

<p align="center">
  <img width="800" alt="image" src="PASTE_SSH_IMAGE_LINK" />
</p>

---

## Key Concepts Demonstrated

- Static Routing Between Two Sites
- Serial Link Configuration (DCE and DTE, Clock Rate)
- One Physical Router Interface Per LAN (Each Interface Acts as the LAN Gateway)
- Switch Management via SVI and Default Gateway
- Shared Local Account with Privilege Level 15 for Console and SSH
- SSHv2 Remote Management
- Enable Secret on All Devices
- Port Security (Sticky MAC, Maximum 1, Shutdown Violation)
- Connectivity Verification with Ping, Tracert, and `show ip route`

---

## What I Learned

Write this part yourself in a few sentences. Ideas: why routes need to exist in both directions, why the DCE side needs the clock rate, why a switch needs a default gateway to be managed from another subnet, what privilege 15 changes about logging in, and what happened when you tested a port security violation.

---

## Skills Practiced

- Static Route Configuration
- Router Interface and Serial Link Configuration
- Switch Management Interface Configuration
- Local User Accounts and Privilege Levels
- SSHv2 and Device Hardening
- Port Security
- Static IP Addressing
- Configuration Verification (`show ip route`, `show ip interface brief`, `show port-security`, `tracert`)
- Saving Configuration to NVRAM
- Cisco Packet Tracer
