# Day 03 - Following Traffic in Simulation Mode (Layers, OSPF, and DHCP)

## Overview

This lab was completed as part of my CCNA studies using Cisco Packet Tracer.

The goal of this lab was to use Packet Tracer's simulation mode to watch how traffic actually moves through a small two router network, and to inspect what each PDU looks like at different layers. I looked at normal frames moving between devices, how data is encapsulated, OSPF traffic between the routers, and the DHCP DORA process when a PC switches from a static address to automatic addressing.

---

## Network Topology

<p align="center">
  <img width="1271" height="514" alt="Day 03 Topology" src="https://github.com/user-attachments/assets/c998de80-15d9-4f17-b936-951610e8cf15" />
</p>

<p align="center">
  Two separate /24 networks joined by two routers. The 192.168.1.0/24 LAN runs from R1 G0/0 out to the switches and end devices on the left side of the topology. The 10.0.0.0/24 network is the link between R1 and R2.
</p>

---

## IP Addressing

| Network | Where it lives | Mask |
|---|---|---|
| 192.168.1.0/24 | R1 G0/0 and all devices on the left side (LAN) | 255.255.255.0 |
| 10.0.0.0/24 | Link between R1 and R2 | 255.255.255.0 |

These are two different subnets that both happen to be /24s. R1 sits in the middle and has an interface in each one, which is what lets it route between them.

---

## Simulation Mode Walkthrough

### Step 1: Watching devices send data to each other

In simulation mode, every packet gets turned into a visible PDU that moves hop by hop across the topology. This makes it easy to see exactly which device touches the traffic and in what order.

<p align="center">
  <img width="750" height="664" alt="Devices sending data" src="https://github.com/user-attachments/assets/855ccdc4-5ec2-4c4b-9a05-5050ff9c1364" />
</p>

### Step 2: Encapsulation across the layers

Clicking on a PDU opens the OSI model view, which shows how the data gets wrapped as it moves down the stack on the sending device and unwrapped on the way back up at the receiver.

<p align="center">
  <img width="653" height="594" alt="Encapsulation layers" src="https://github.com/user-attachments/assets/1997c955-1f32-4c4c-a923-bc7669db6574" />
</p>

Quick refresher on what gets added at each step:

| Layer | PDU name | What gets added |
|---|---|---|
| Application / Transport | Segment | Port numbers (TCP or UDP header) |
| Network | Packet | Source and destination IP address |
| Data Link | Frame | Source and destination MAC address |
| Physical | Bits | Signal on the wire |

### Step 3: Layer 3 traffic

Next I looked at traffic that includes Layer 3 information.

<p align="center">
  <img width="1566" height="657" alt="Layer 3 traffic" src="https://github.com/user-attachments/assets/45132357-bcea-415a-b7b8-db7955a4e7fc" />
</p>

Here we can see OSPF traffic. OSPF is the routing protocol the routers use to share information about the networks they know and work out the best path to reach each one, which is how routers learn routes when there are multiple networks involved.

<p align="center">
  <img width="1415" height="788" alt="OSPF traffic" src="https://github.com/user-attachments/assets/db1e5163-5fa2-4fe3-9314-aa69a4df2bd3" />
</p>

A few things worth noting about OSPF packets:

- They carry Layer 3 information, meaning source and destination **IP addresses**
- They still sit inside an Ethernet frame, so there are **MAC addresses** at Layer 2 too. IP does not replace the MAC address, both exist at the same time
- OSPF messages are sent to a multicast address (224.0.0.5 for all OSPF routers) so every OSPF router on the link receives them
- OSPF runs directly on top of IP (protocol number 89), so there is no TCP or UDP port number involved

### Step 4: DHCP and the DORA process

To look at traffic that touches every layer, I used DHCP. DHCP is how devices automatically get an IP address from a pool (scope) on the network instead of being set by hand.

<p align="center">
  <img width="1395" height="538" alt="DHCP DORA process" src="https://github.com/user-attachments/assets/7abfe545-a581-41df-b9b9-ed340b8b1bad" />
</p>

After switching the PC from a static address to DHCP, it went through the full **DORA** process and successfully received an IP address:

| Step | Message | What happens |
|---|---|---|
| D | Discover | Client broadcasts looking for any DHCP server |
| O | Offer | Server offers an IP address from its scope |
| R | Request | Client asks to use the offered address |
| A | Acknowledge | Server confirms and the client can start using it |

DHCP runs over UDP, using port 67 on the server and port 68 on the client.

### A note on Layers 5 and 6

I did not see separate Layer 5 (Session) or Layer 6 (Presentation) information in the PDU details. That is because the TCP/IP model combines the top three OSI layers (Session, Presentation, and Application) into one Application layer, so those functions are handled by the application protocol itself, DHCP in this case.

---

## Key Concepts Demonstrated

- Packet Tracer Simulation Mode
- Encapsulation and De-encapsulation
- OSI Model vs TCP/IP Model
- Layer 2 vs Layer 3 Addressing (MAC vs IP)
- OSPF Basics (Layer 3 Routing Protocol Traffic)
- DHCP DORA Process
- Multiple Subnets Connected Through Routers

---

## What I Learned

Seeing the traffic move one hop at a time made the layers a lot less abstract. I could actually see the IP and MAC addresses in the same PDU and understand that a packet carries both at once, each doing a different job. OSPF showed me that routers talk to each other with their own traffic, not just user traffic, and watching DORA play out made it clear how a device with no address finds a server and gets one. It also helped me understand why the TCP/IP model merges the top OSI layers, since the application protocol handles all of that on its own.

---

## Skills Practiced

- Packet Tracer Simulation Mode and PDU Inspection
- Reading Encapsulation Across OSI Layers
- Identifying Layer 2 and Layer 3 Information in Traffic
- Recognizing OSPF and DHCP Traffic
- Understanding the DHCP DORA Process
- Cisco Packet Tracer

---

## Verification

- Traffic moved between devices on the 192.168.1.0/24 LAN and across to the 10.0.0.0/24 link in simulation mode
- OSPF packets were visible between R1 and R2 with Layer 3 information in the PDU
- After switching the PC from static to DHCP, the full DORA exchange completed and the PC received an address from the scope
