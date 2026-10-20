# Building a Secure Office Network with VLANs and Router-on-a-Stick

*How I segmented HR, IT, and Finance into separate networks with Cisco Packet Tracer.*

A small office network can appear simple at first: connect every computer to a switch, give each device an IP address, and allow everyone to communicate. The problem is that this design puts every device in the same broadcast domain. HR systems, IT workstations, and finance records all become part of one large flat network.

For this project, I designed a more structured office network using **VLANs**, **802.1Q trunks**, and **Router-on-a-Stick** inter-VLAN routing in Cisco Packet Tracer. The goal was to improve organization, reduce unnecessary broadcasts, and provide logical separation between departments without requiring a separate physical switch for every department.

> **Project repository:** [Design and Configuration of an Office Network Using VLANs](https://github.com/gene707/Design-and-Configuration-Of-an-Office-Network-using-VLANs)

![Configured office VLAN topology](Proof%20of%20Concept/02-network-topology.png)

## The business problem: one office, three departments

The office has three departments with different responsibilities:

- **Human Resources (HR)** manages employee records and recruitment.
- **Information Technology (IT)** manages technical operations and network infrastructure.
- **Finance** manages financial records and transactions.

Placing all three departments in one network is difficult to manage and creates unnecessary broadcast traffic. It also provides little logical separation between departments with very different data and responsibilities.

The solution is VLAN segmentation. A VLAN creates a separate Layer 2 broadcast domain, even when devices are connected to the same physical switching infrastructure.

## The design

The topology uses:

- One Cisco router
- Two Cisco 2960 switches
- Six PCs—two for each department
- One router-to-switch trunk
- One switch-to-switch trunk

Each department is distributed across the two switches. This is important because it proves that a VLAN is not tied to one physical switch or one location. A VLAN can extend across multiple switches as long as trunk links carry it.

| Department | VLAN | Network | Default gateway |
| --- | ---: | --- | --- |
| HR | 10 | 192.168.10.0/24 | 192.168.10.1 |
| IT | 20 | 192.168.20.0/24 | 192.168.20.1 |
| Finance | 30 | 192.168.30.0/24 | 192.168.30.1 |

## Step 1: Creating department VLANs

I created the same VLANs on both switches:

```cisco
vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name FINANCE
```

After creating the VLANs, I assigned the PC-facing ports as access ports. For example, an HR PC port is configured as follows:

```cisco
interface fastethernet0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
```

The same process was used for the IT and Finance access ports, with the appropriate VLAN number.

![Switch VLAN configuration](Proof%20of%20Concept/01-switch-vlan-configuration.png)

### Why access ports matter

An access port carries traffic for one VLAN. The PC does not need to understand VLAN tags; the switch associates all incoming untagged traffic from that port with the configured VLAN.

This means an HR workstation connected to a VLAN 10 access port is automatically placed in the HR broadcast domain.

## Step 2: Carrying multiple VLANs over trunk links

The two switches must share all three department VLANs. The router must also receive all three VLANs so it can route between their IP subnets. This is why both inter-device links are configured as trunks.

```cisco
interface gigabitethernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

An 802.1Q trunk adds a VLAN tag to frames crossing the trunk. That tag tells the receiving device whether a frame belongs to VLAN 10, 20, or 30.

In this design:

- The switch-to-switch trunk transports department traffic between the two switches.
- The router-to-switch trunk transports traffic to and from the router’s VLAN subinterfaces.

This is the key reason a department can have one PC on Switch 1 and another on Switch 2 while still behaving as one logical LAN.

## Step 3: Configuring Router-on-a-Stick

VLANs separate Layer 2 networks. A router is required when devices in different VLANs need to communicate.

Instead of using one physical router interface per VLAN, Router-on-a-Stick uses one physical interface with multiple logical subinterfaces. Each subinterface receives traffic for one 802.1Q VLAN and provides that VLAN’s default gateway.

```cisco
interface gigabitethernet0/0
 no ip address
 no shutdown

interface gigabitethernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gigabitethernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface gigabitethernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

Each subinterface is the default gateway for devices in its VLAN. For example, HR devices use `192.168.10.1`, while Finance devices use `192.168.30.1`.

![Router trunk interface details](Proof%20of%20Concept/05-router-trunk-interface.png)

## Step 4: Assigning endpoint addressing

Every PC received a static IPv4 address, subnet mask, and the gateway for its VLAN. The Finance PC example below uses the Finance subnet and router subinterface as its gateway.

![Finance PC static addressing](Proof%20of%20Concept/03-finance-pc-ip-configuration.png)

For example:

```text
Finance PC IP address: 192.168.30.11
Subnet mask:          255.255.255.0
Default gateway:      192.168.30.1
```

Correct default-gateway configuration is essential. A PC sends same-subnet traffic directly to the destination PC, but sends traffic for another subnet to the default gateway.

## How the packet moves between VLANs

A ping from an HR PC (`192.168.10.11`) to a Finance PC (`192.168.30.11`) demonstrates the full process:

1. HR-PC1 compares the destination address with its own subnet mask and identifies the Finance PC as remote.
2. HR-PC1 uses ARP to discover the MAC address of its default gateway, `192.168.10.1`.
3. It sends the packet to the switch on VLAN 10.
4. The switch forwards the frame across the router trunk with an 802.1Q VLAN 10 tag.
5. The router receives the frame on `G0/0.10`, removes the incoming Ethernet header, and examines the destination IP address.
6. The routing table selects `G0/0.30` because `192.168.30.0/24` is directly connected there.
7. The router builds a new Ethernet frame for VLAN 30, using its outgoing MAC address as the source and the Finance PC’s MAC address as the destination.
8. The frame crosses the trunk with a VLAN 30 tag and reaches the Finance PC through its VLAN 30 access port.

The source and destination **IP addresses remain unchanged** during ordinary routing without NAT. The Ethernet source and destination **MAC addresses change** at the router because the router creates a new Layer 2 frame for the next network.

## Verification and troubleshooting

The most useful verification commands were:

```cisco
show vlan brief
show interfaces trunk
show interfaces status
show mac address-table
show ip interface brief
show ip route
show running-config
```

These commands answer different questions:

- Is each PC port in the correct VLAN?
- Are both trunks active and carrying VLANs 10, 20, and 30?
- Are router subinterfaces up and addressed correctly?
- Does the router have connected routes to all three networks?
- Has the switch learned endpoint MAC addresses in the expected VLAN?

One of the most valuable tests was disabling the router-facing trunk temporarily. Same-VLAN devices could still communicate through the switches, but devices in different VLANs could not. This clearly shows the distinction between Layer 2 switching and Layer 3 routing.

![Saved switch configuration](Proof%20of%20Concept/04-switch-configuration-saved.png)

## Key lessons from the project

### VLANs create logical separation

VLANs do not require every department to have its own physical switch. They allow departments to remain separate while sharing switching hardware.

### Trunks extend VLANs across the network

802.1Q trunking is what allows VLAN 10, 20, and 30 to travel between switches and reach the router.

### Inter-VLAN traffic needs Layer 3 routing

A switch can forward same-VLAN traffic, but it cannot route traffic between different IP networks without Layer 3 capabilities. Router-on-a-Stick provides that function using subinterfaces.

### Troubleshooting should follow the packet path

When a ping fails, checking from Layer 1 upward is reliable:

1. Is the cable/interface up?
2. Is the PC port in the correct VLAN?
3. Is the VLAN present on both switches?
4. Is the trunk active and allowing the VLAN?
5. Is the router subinterface configured with the matching dot1Q tag and gateway address?
6. Does the PC have the correct address, mask, and default gateway?

## Final thoughts

This project demonstrates why VLANs are a foundational networking skill. They improve organization, reduce broadcast scope, and provide meaningful isolation between groups of users. Router-on-a-Stick then enables traffic to move between those groups when communication is needed.

The project can be extended by adding access control lists to restrict traffic between departments, assigning a management VLAN, deploying DHCP, or replacing Router-on-a-Stick with a Layer 3 switch for greater performance in larger environments.

---

## Explore the project

- **Repository and full setup guide:** [Design and Configuration of an Office Network Using VLANs](https://github.com/gene707/Design-and-Configuration-Of-an-Office-Network-using-VLANs)
- **Packet Tracer file:** [Office_VLAN_Routing_Project.pkt](Proof%20of%20Concept/Office_VLAN_Routing_Project.pkt)
- **Screenshots and configuration evidence:** [Proof of Concept](Proof%20of%20Concept/)

*What is the first improvement you would make to this office VLAN design: ACLs, DHCP, a management VLAN, or Layer 3 switching?*
