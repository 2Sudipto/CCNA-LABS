# Day XX - Static Routing Between Two Sites (NY and NJ)

## Overview

This lab was completed as part of my CCNA studies using Cisco Packet Tracer.

The purpose of this lab is to connect two sites, NY and NJ, over a serial WAN link and use static routes so every LAN can reach every other LAN. Each site has two routed LANs behind a router and a switch per LAN. On top of the routing, the lab also covers basic hardening: SSHv2 for remote management, console login, and port security on the access ports.

---

## Network Topology

<p align="center">
  <img width="900" alt="image" src="PASTE_TOPOLOGY_IMAGE_LINK" />
</p>

<p align="center">
  Router0 (NY) and Router1 (NJ) are connected by a 1.1.1.0/30 serial link. Each router has two LAN-facing interfaces, and each LAN has one switch with two PCs. Router0 is the DCE side of the serial link.
</p>

---

## IP Addressing

### Routers

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Router0 | Gi0/0/0 | 192.168.1.1 | 255.255.255.0 |
| Router0 | Gi0/0/1 | 192.168.2.1 | 255.255.255.0 |
| Router0 | S0/1/0 (DCE) | 1.1.1.1 | 255.255.255.252 |
| Router1 | Gi0/0/0 | 192.168.3.1 | 255.255.255.0 |
| Router1 | Gi0/0/1 | 192.168.4.1 | 255.255.255.0 |
| Router1 | S0/1/0 | 1.1.1.2 | 255.255.255.252 |

### Switches and PCs

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| Switch0 | VLAN 1 | 192.168.1.254 | 255.255.255.0 | 192.168.1.1 |
| Switch1 | VLAN 1 | 192.168.2.254 | 255.255.255.0 | 192.168.2.1 |
| Switch2 | VLAN 1 | 192.168.3.254 | 255.255.255.0 | 192.168.3.1 |
| Switch3 | VLAN 1 | 192.168.4.254 | 255.255.255.0 | 192.168.4.1 |
| PC0 | NIC | 192.168.1.X | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.X | 255.255.255.0 | 192.168.1.1 |
| PC0(1) | NIC | 192.168.2.X | 255.255.255.0 | 192.168.2.1 |
| PC1(1) | NIC | 192.168.2.X | 255.255.255.0 | 192.168.2.1 |
| PC0(2) | NIC | 192.168.3.X | 255.255.255.0 | 192.168.3.1 |
| PC1(2) | NIC | 192.168.3.X | 255.255.255.0 | 192.168.3.1 |
| PC1(3) | NIC | 192.168.4.X | 255.255.255.0 | 192.168.4.1 |
| PC0(3) | NIC | 192.168.4.X | 255.255.255.0 | 192.168.4.1 |

---

## Configuration Steps

### Configure Hostname, Passwords, and Banner on Router0
```
Router>enable
Router#configure terminal
Router(config)#hostname Router0
Router0(config)#no ip domain-lookup
Router0(config)#enable secret cisco
Router0(config)#service password-encryption
Router0(config)#banner motd #Authorized access only. Violators will be prosecuted to the full extent of the law.#
```

`no ip domain-lookup` stops the router from trying to resolve mistyped commands as hostnames, which saves a lot of waiting around. `service password-encryption` makes sure any plain text line passwords in the config get obscured.

### Configure the Router Interfaces
```
Router0(config)#interface gigabitEthernet 0/0/0
Router0(config-if)#ip address 192.168.1.1 255.255.255.0
Router0(config-if)#no shutdown
Router0(config-if)#exit
Router0(config)#interface gigabitEthernet 0/0/1
Router0(config-if)#ip address 192.168.2.1 255.255.255.0
Router0(config-if)#no shutdown
Router0(config-if)#exit
Router0(config)#interface serial 0/1/0
Router0(config-if)#ip address 1.1.1.1 255.255.255.252
Router0(config-if)#clock rate 64000
Router0(config-if)#no shutdown
```

Router0 is the DCE side of the serial link, so it is the one that needs `clock rate`. Without it the link never comes up. Router interfaces are also administratively down by default, so every one of them needs a `no shutdown`.

### Repeat on Router1
Same hostname, password, and banner steps with `hostname Router1`. Use the Router1 addresses from the table above. Do not add `clock rate` on Router1, since it is the DTE side.

### Save the Configuration
```
Router0#copy running-config startup-config
```

---

### Configure the Switch Management Interface and Gateway
```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Switch0
Switch0(config)#enable secret cisco
Switch0(config)#interface vlan 1
Switch0(config-if)#ip address 192.168.1.254 255.255.255.0
Switch0(config-if)#no shutdown
Switch0(config-if)#exit
Switch0(config)#ip default-gateway 192.168.1.1
```

The default gateway matters here. Without it the switch can only be reached from devices on its own subnet, so SSHing into Switch0 from the NJ side would fail even with perfect routing.

Repeat on Switch1, Switch2, and Switch3 using their own addresses from the table.

### Configure the PCs
On each PC, open the Desktop tab, go to IP Configuration, and enter the static IP, subnet mask, and default gateway from the addressing table.

<p align="center">
  <img width="800" alt="image" src="PASTE_PC_CONFIG_IMAGE_LINK" />
</p>

### Test Local Connectivity First
From PC0's Command Prompt:
```
PC> ping 192.168.1.1
PC> ping 192.168.1.254
```

Both should succeed. Then from Router0:
```
Router0#ping 1.1.1.2
```

This should also succeed, which confirms the serial link is up before touching routing. If it fails, check `clock rate` on the DCE side and `show ip interface brief` on both routers.

<p align="center">
  <img width="800" alt="image" src="PASTE_LOCAL_PING_IMAGE_LINK" />
</p>

---

### Test Cross-Site Connectivity Before Static Routes
From PC0 in NY:
```
PC> ping 192.168.3.X
```

This is expected to fail. Router0 only knows about its directly connected networks, so it has no idea where 192.168.3.0/24 lives. Running `show ip route` on Router0 at this point shows only `C` and `L` entries.

<p align="center">
  <img width="800" alt="image" src="PASTE_PING_FAIL_IMAGE_LINK" />
  <img width="800" alt="image" src="PASTE_ROUTE_TABLE_BEFORE_IMAGE_LINK" />
</p>

### Configure the Static Routes
On Router0 (NY), point to the NJ LANs through Router1's serial address:
```
Router0(config)#ip route 192.168.3.0 255.255.255.0 1.1.1.2
Router0(config)#ip route 192.168.4.0 255.255.255.0 1.1.1.2
```

On Router1 (NJ), point to the NY LANs through Router0's serial address:
```
Router1(config)#ip route 192.168.1.0 255.255.255.0 1.1.1.1
Router1(config)#ip route 192.168.2.0 255.255.255.0 1.1.1.1
```

Routes have to exist in both directions. A ping needs a path there and a path back, so if only one router has routes, the echo request arrives but the reply gets dropped.

### Verify the Routing Table
```
Router0#show ip route
```

The new routes should show up with an `S` code.

<p align="center">
  <img width="800" alt="image" src="PASTE_ROUTE_TABLE_AFTER_IMAGE_LINK" />
</p>

### Verify Cross-Site Connectivity
From PC0 in NY:
```
PC> ping 192.168.3.X
PC> ping 192.168.4.X
```

From a PC in NJ:
```
PC> ping 192.168.1.X
PC> ping 192.168.2.X
```

All pings should succeed. A first attempt sometimes shows a dropped packet, which is normal and comes from ARP resolution on the first packet.

<p align="center">
  <img width="800" alt="image" src="PASTE_PING_SUCCESS_IMAGE_LINK" />
</p>

---

### Configure SSHv2 for Remote Management
```
Router0(config)#ip domain-name lab.local
Router0(config)#username sshadmin secret sshpw
Router0(config)#crypto key generate rsa modulus 2048
Router0(config)#ip ssh version 2
Router0(config)#line vty 0 15
Router0(config-line)#login local
Router0(config-line)#transport input ssh
Router0(config-line)#exit
```

SSH needs a hostname and a domain name before it can generate the RSA keys, which is why those come first. `transport input ssh` blocks Telnet, so anything sent to the vty lines is encrypted.

Repeat on every router and switch.

### Configure Console Login
```
Router0(config)#username admin secret consolepw
Router0(config)#line console 0
Router0(config-line)#login local
Router0(config-line)#exit
```

### Configure Port Security on the Switches
Apply this to the PC-facing access ports only, not the uplink to the router.
```
Switch0(config)#interface range fastEthernet 0/1 - 2
Switch0(config-if-range)#switchport mode access
Switch0(config-if-range)#switchport port-security
Switch0(config-if-range)#switchport port-security maximum 1
Switch0(config-if-range)#switchport port-security violation shutdown
Switch0(config-if-range)#switchport port-security mac-address sticky
```

Sticky learns the MAC address of the first device that sends traffic on the port and saves it to the running config. With a max of 1 and violation set to shutdown, plugging a different device into that port puts it in err-disabled state.

### Verify Port Security and SSH
```
Switch0#show port-security
Switch0#show port-security interface fastEthernet 0/1
```

From a PC in the other site:
```
PC> ssh -l sshadmin 192.168.1.254
```

<p align="center">
  <img width="800" alt="image" src="PASTE_PORT_SECURITY_IMAGE_LINK" />
  <img width="800" alt="image" src="PASTE_SSH_IMAGE_LINK" />
</p>

### Save the Configuration on Every Device
```
Router0#copy running-config startup-config
```

---

## Key Concepts Demonstrated

- Static Routing Between Two Sites
- Serial Link Configuration (DCE and DTE, Clock Rate)
- One Physical Router Interface Per LAN (Each Interface Acts as the LAN Gateway)
- Switch Management via SVI and Default Gateway
- SSHv2 Remote Management
- Console and Privileged EXEC Password Security
- Port Security (Sticky MAC, Maximum 1, Shutdown Violation)
- Connectivity Verification with Ping and `show ip route`

---

## What I Learned

Write this part yourself in a few sentences. Ideas: why routes need to exist in both directions, why the DCE side needs the clock rate, why a switch needs a default gateway to be managed from another subnet, and what happened when you tested a port security violation.

---

## Skills Practiced

- Static Route Configuration
- Router Interface and Serial Link Configuration
- Switch Management Interface Configuration
- SSHv2 and Device Hardening
- Port Security
- Static IP Addressing
- Configuration Verification (`show ip route`, `show ip interface brief`, `show port-security`)
- Saving Configuration to NVRAM
- Cisco Packet Tracer
