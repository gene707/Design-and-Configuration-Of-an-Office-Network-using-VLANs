# Design and Configuration of an Office Network Using VLANs

> **Cisco Packet Tracer project:** VLAN segmentation, 802.1Q trunking, and Router-on-a-Stick inter-VLAN routing.

## Project overview

This project builds a small office network for three departments:

- **Human Resources (HR)** — VLAN 10
- **Information Technology (IT)** — VLAN 20
- **Finance** — VLAN 30

Each department is placed in its own logical network. Two Cisco 2960 switches carry the VLANs, and one router provides inter-VLAN routing through a single 802.1Q trunk link (Router-on-a-Stick).

## Objectives

- Separate HR, IT, and Finance users into distinct VLANs.
- Reduce unnecessary broadcast traffic by creating independent broadcast domains.
- Carry all department VLANs between the switches and to the router through trunk links.
- Route traffic between VLANs with router subinterfaces.
- Verify both same-VLAN and inter-VLAN connectivity.
- Observe ARP, VLAN tagging, and MAC-address changes in Packet Tracer Simulation Mode.

---

## Required devices and topology

### Devices

| Device | Quantity | Suggested model |
| --- | ---: | --- |
| Router | 1 | Any router with a Gigabit Ethernet interface that supports 802.1Q subinterfaces (for example, Cisco 1941/2911) |
| Switch | 2 | Cisco 2960 |
| HR PCs | 2 | PC-PT |
| IT PCs | 2 | PC-PT |
| Finance PCs | 2 | PC-PT |
| Server | 1 (optional) | Server-PT |

### Logical topology

```text
                  802.1Q trunk
              Router G0/0
                    |
                    |
              Switch 1 ---------------- Switch 2
                         802.1Q trunk

       HR-PC1, IT-PC1, FIN-PC1       HR-PC2, IT-PC2, FIN-PC2
```

The PCs must be distributed across both switches. The example port assignment below places one PC from every department on each switch.

### Example cabling and port plan

Use copper straight-through cables for PC-to-switch and router-to-switch links. Use the Packet Tracer automatic connection tool or a suitable Ethernet cable for the switch-to-switch link.

| Device | Interface | Connected to | Purpose |
| --- | --- | --- | --- |
| Router | G0/0 | Switch 1 G0/1 | Router-to-switch trunk |
| Switch 1 | G0/2 | Switch 2 G0/1 | Switch-to-switch trunk |
| Switch 1 | F0/1 | HR-PC1 | VLAN 10 access port |
| Switch 1 | F0/2 | IT-PC1 | VLAN 20 access port |
| Switch 1 | F0/3 | FIN-PC1 | VLAN 30 access port |
| Switch 2 | F0/1 | HR-PC2 | VLAN 10 access port |
| Switch 2 | F0/2 | IT-PC2 | VLAN 20 access port |
| Switch 2 | F0/3 | FIN-PC2 | VLAN 30 access port |

> If your router or switch uses different interface names, substitute the correct names throughout the commands. For example, use `FastEthernet0/0` instead of `GigabitEthernet0/0` when required.

---

## VLAN and addressing plan

| VLAN | VLAN name | Department | Network | Default gateway |
| ---: | --- | --- | --- | --- |
| 10 | HR | Human Resources | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT | Information Technology | 192.168.20.0/24 | 192.168.20.1 |
| 30 | FINANCE | Finance | 192.168.30.0/24 | 192.168.30.1 |

| Device | VLAN | IP address | Subnet mask | Default gateway |
| --- | ---: | --- | --- | --- |
| HR-PC1 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| HR-PC2 | 10 | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 |
| IT-PC1 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| IT-PC2 | 20 | 192.168.20.12 | 255.255.255.0 | 192.168.20.1 |
| FIN-PC1 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |
| FIN-PC2 | 30 | 192.168.30.12 | 255.255.255.0 | 192.168.30.1 |

---

# Step-by-step Packet Tracer setup

## 1. Build the physical topology

1. Open **Cisco Packet Tracer** and create a new logical workspace.
2. Add one compatible router, two **2960** switches, and six **PC-PT** devices.
3. Rename the devices clearly: `R1`, `S1`, `S2`, `HR-PC1`, `HR-PC2`, `IT-PC1`, `IT-PC2`, `FIN-PC1`, and `FIN-PC2`.
4. Connect `R1 G0/0` to `S1 G0/1`.
5. Connect `S1 G0/2` to `S2 G0/1`.
6. Connect the PCs to the access ports in the example cabling plan.
7. Wait for link lights to become active. An initially amber switch port may take a short time to transition because of spanning tree.

## 2. Create VLANs on both switches

Perform this configuration on **S1** and then repeat it on **S2**.

```cisco
enable
configure terminal

vlan 10
 name HR
exit
vlan 20
 name IT
exit
vlan 30
 name FINANCE
exit

end
copy running-config startup-config
```

Verify the VLAN database:

```cisco
show vlan brief
```

Expected VLANs: `10 HR`, `20 IT`, and `30 FINANCE`.

## 3. Assign switch access ports to the correct VLANs

### Configure S1 access ports

```cisco
enable
configure terminal

interface fastethernet0/1
 description HR-PC1 - VLAN 10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface fastethernet0/2
 description IT-PC1 - VLAN 20
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface fastethernet0/3
 description FIN-PC1 - VLAN 30
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

end
copy running-config startup-config
```

### Configure S2 access ports

```cisco
enable
configure terminal

interface fastethernet0/1
 description HR-PC2 - VLAN 10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface fastethernet0/2
 description IT-PC2 - VLAN 20
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface fastethernet0/3
 description FIN-PC2 - VLAN 30
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

end
copy running-config startup-config
```

> `spanning-tree portfast` is appropriate only for end-device access ports. Do **not** enable it on links between network devices.

## 4. Configure trunk links

The trunks must transport VLANs 10, 20, and 30. On Cisco 2960 switches, `switchport trunk encapsulation dot1q` is usually not available because 802.1Q is the fixed encapsulation.

### S1 trunks

```cisco
enable
configure terminal

interface gigabitethernet0/1
 description 802.1Q trunk to R1 G0/0
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface gigabitethernet0/2
 description 802.1Q trunk to S2 G0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

end
copy running-config startup-config
```

### S2 trunk

```cisco
enable
configure terminal

interface gigabitethernet0/1
 description 802.1Q trunk to S1 G0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

end
copy running-config startup-config
```

Verify each switch:

```cisco
show interfaces trunk
show interfaces status
```

Both sides of the switch-to-switch link should be trunking, and VLANs 10, 20, and 30 should appear as allowed and active.

## 5. Configure Router-on-a-Stick on R1

The router interface connected to S1 is a physical trunk carrier. It has **no IP address itself**; IP addresses belong on the VLAN subinterfaces.

```cisco
enable
configure terminal

interface gigabitethernet0/0
 description 802.1Q trunk to S1 G0/1
 no ip address
 no shutdown
exit

interface gigabitethernet0/0.10
 description Default gateway for HR (VLAN 10)
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitethernet0/0.20
 description Default gateway for IT (VLAN 20)
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

interface gigabitethernet0/0.30
 description Default gateway for FINANCE (VLAN 30)
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit

end
copy running-config startup-config
```

Verify R1:

```cisco
show ip interface brief
show ip route
show running-config
show interfaces
```

The router should show `G0/0.10`, `G0/0.20`, and `G0/0.30` as up/up when all links are active. Its routing table should contain connected routes for `192.168.10.0/24`, `192.168.20.0/24`, and `192.168.30.0/24`.

## 6. Configure the PCs

For each PC:

1. Select the PC.
2. Open **Desktop > IP Configuration**.
3. Choose **Static**.
4. Enter the address, subnet mask, and default gateway from the addressing plan.

Example for `HR-PC1`:

```text
IP address:      192.168.10.11
Subnet mask:     255.255.255.0
Default gateway: 192.168.10.1
```

Repeat for all six PCs using their assigned values.

## 7. Save the Packet Tracer file

Save the completed file using the required naming convention:

```text
StudentName_VLAN_Routing_Project.pkt
```

Replace `StudentName` with your own name.

---

# Testing and verification

## Test A — Same-VLAN communication

Run these commands in each source PC under **Desktop > Command Prompt**:

```text
HR-PC1> ping 192.168.10.12
IT-PC1> ping 192.168.20.12
FIN-PC1> ping 192.168.30.12
```

**Expected result:** all pings succeed. The first ICMP echo can time out while ARP resolves MAC addresses; repeat the ping if necessary.

## Test B — Inter-VLAN communication

```text
HR-PC1> ping 192.168.20.11
HR-PC1> ping 192.168.30.11
FIN-PC1> ping 192.168.10.11
```

**Expected result:** all pings succeed after the VLANs, access ports, trunks, router subinterfaces, and PC gateways are configured correctly.

## Test C — VLAN isolation with the router link disabled

Temporarily shut down the router-facing trunk on S1:

```cisco
S1# configure terminal
S1(config)# interface gigabitethernet0/1
S1(config-if)# shutdown
S1(config-if)# end
```

Then test one same-VLAN ping and one inter-VLAN ping.

### Answers

1. **Can two PCs in the same VLAN still communicate?**

   **Yes.** Same-VLAN devices can communicate at Layer 2 through the switches and the switch-to-switch trunk, so the router is not required for that traffic.

2. **Can a PC in VLAN 10 communicate with a PC in VLAN 20?**

   **No.** VLAN 10 and VLAN 20 are separate Layer 2 broadcast domains and IP subnets. Their traffic needs a Layer 3 device to route between them.

3. **Why does router availability affect different-VLAN communication but not same-VLAN communication?**

   Hosts in the same VLAN use local Ethernet switching after resolving the destination MAC address. For a remote subnet, a host sends the frame to its default gateway. The router receives the packet on one VLAN subinterface and routes it out through another VLAN subinterface. With the router link down, this gateway path is unavailable.

Restore the router trunk when finished:

```cisco
S1# configure terminal
S1(config)# interface gigabitethernet0/1
S1(config-if)# no shutdown
S1(config-if)# end
S1# copy running-config startup-config
```

---

# Simulation Mode analysis

Switch Packet Tracer from **Realtime** to **Simulation** mode, filter for ARP and ICMP if needed, and send a ping from `HR-PC1 (192.168.10.11)` to `FIN-PC1 (192.168.30.11)`.

## 1. How does the source PC identify the default gateway MAC address?

Because `192.168.30.11` is outside HR-PC1's local `192.168.10.0/24` subnet, HR-PC1 sends traffic to its configured default gateway, `192.168.10.1`. If its ARP cache does not contain the gateway's MAC address, HR-PC1 sends an **ARP broadcast** in VLAN 10 asking, “Who has 192.168.10.1?” R1's `G0/0.10` subinterface replies with its MAC address. HR-PC1 then sends the ICMP packet in an Ethernet frame addressed to the router's VLAN 10 gateway MAC address.

## 2. How does the Ethernet frame travel from the access switch to the router?

HR-PC1 sends an untagged Ethernet frame into its VLAN 10 access port on S1. S1 recognizes the port as VLAN 10 and forwards the frame toward the router trunk. On the trunk, the frame is carried with an **802.1Q VLAN 10 tag**. The router receives it on subinterface `G0/0.10`, which matches that tag.

## 3. How does the router identify the correct outgoing interface?

R1 removes the incoming Layer 2 frame and examines the destination IP address, `192.168.30.11`. Its routing table has a directly connected route for `192.168.30.0/24` through `G0/0.30`, so R1 selects that subinterface as the outgoing interface. If necessary, it uses ARP in VLAN 30 to learn FIN-PC1's MAC address.

## 4. What happens to the Ethernet source and destination MAC addresses when the router forwards the packet?

The router discards the incoming Ethernet header and creates a **new Ethernet frame** for VLAN 30. The outgoing source MAC address is R1's MAC address on the VLAN 30 side, and the destination MAC address is FIN-PC1's MAC address. MAC addresses therefore change at each routed hop.

## 5. Do the source and destination IP addresses change during ordinary routing without NAT?

**No.** Without NAT, the IP source remains `192.168.10.11` and the IP destination remains `192.168.30.11`. The router decrements the IP TTL by one and recalculates the IPv4 header checksum, but it does not change the source or destination IP address.

---

# Required command reference

Include command output or screenshots in the final report where requested.

## Switch commands

| Command | Information provided |
| --- | --- |
| `show vlan brief` | Lists VLAN IDs, names, status, and access ports assigned to each VLAN. Use it to confirm VLAN 10/20/30 and PC port membership. |
| `show interfaces trunk` | Shows interfaces operating as trunks, their encapsulation, native VLAN, allowed VLANs, and VLANs currently forwarding. Use it to verify both trunks carry VLANs 10, 20, and 30. |
| `show interfaces status` | Summarizes interface state, VLAN/trunk mode, duplex, speed, and port type. Use it to check whether ports are connected and whether access ports belong to expected VLANs. |
| `show mac address-table` | Displays MAC addresses learned by the switch, associated VLANs, ports, and entry types. Use it to confirm endpoints are learned in the correct VLAN. |
| `show running-config` | Displays the active switch configuration, including VLAN access-port assignments and trunk commands. |

## Router commands

| Command | Information provided |
| --- | --- |
| `show ip interface brief` | Quickly lists router interfaces, assigned IP addresses, and line/protocol status. Use it to confirm the physical interface and all three subinterfaces are up/up. |
| `show ip route` | Displays the routing table. It should show connected networks 192.168.10.0/24, 192.168.20.0/24, and 192.168.30.0/24. |
| `show running-config` | Displays the active Router-on-a-Stick configuration, including `encapsulation dot1Q` and the gateway IP address for each subinterface. |
| `show interfaces` | Provides detailed per-interface statistics and status, useful for checking errors, encapsulation, MAC addresses, traffic counters, and link operation. |

---

# Troubleshooting report

| Fault / symptom | Likely cause | Diagnosis | Solution |
| --- | --- | --- | --- |
| A PC cannot ping its same-VLAN partner on the other switch. | The switch-to-switch link is not trunking or does not allow the VLAN. | Run `show interfaces trunk` on both switches. | Configure both ends as trunks and allow VLANs `10,20,30`. |
| A PC cannot ping its gateway. | Wrong PC IP/gateway, wrong access VLAN, router physical interface is down, or router trunk is misconfigured. | Check PC Desktop IP Configuration; use `show vlan brief`, `show ip interface brief`, and `show interfaces trunk`. | Correct the addressing/VLAN assignment, enable the router interface, and repair the trunk. |
| Devices in different VLANs cannot ping each other, but same-VLAN pings work. | Router subinterfaces are missing, have incorrect `encapsulation dot1Q` VLAN numbers, or the router trunk does not carry all VLANs. | Use `show ip interface brief`, `show running-config`, `show ip route`, and `show interfaces trunk`. | Create/correct `.10`, `.20`, `.30` subinterfaces; match dot1Q tags to VLAN IDs; permit all VLANs on the trunk. |
| Devices connected to a port cannot communicate with the intended department. | The access port is in the wrong VLAN or is in the default VLAN 1. | Run `show vlan brief` and `show interfaces status`. | Apply `switchport mode access` and `switchport access vlan <10|20|30>` to the correct port. |
| The trunk exists but a specific VLAN does not cross it. | VLAN is absent from the allowed list or was not created on the downstream switch. | Run `show interfaces trunk` and `show vlan brief` on both switches. | Create the VLAN on both switches and use `switchport trunk allowed vlan 10,20,30`. |
| The first ping fails but later pings succeed. | ARP resolution is occurring. | Observe ARP packets in Simulation Mode. | This is normal; send the ping again after ARP has populated the cache. |
| A link remains down/down. | Incorrect cable, wrong interface, or an administratively shut interface. | Use `show interfaces status` or `show ip interface brief`. | Check cabling and port selection, then apply `no shutdown` where needed. |

---

# Required evidence checklist

The `.pkt` file is submitted separately. Before submission, collect the following evidence:

- [ ] Completed Packet Tracer file named `StudentName_VLAN_Routing_Project.pkt`.
- [ ] Screenshot of the complete network topology.
- [ ] `show vlan brief` screenshots from **both** switches.
- [ ] `show interfaces trunk` screenshots showing the switch-to-switch and router-to-switch trunks.
- [ ] Router `show ip interface brief` / `show running-config` screenshots showing `G0/0.10`, `G0/0.20`, and `G0/0.30`.
- [ ] Successful same-VLAN ping results.
- [ ] Successful inter-VLAN ping results.
- [ ] Simulation Mode explanation (provided above).
- [ ] Troubleshooting report (provided above).
- [ ] Conclusion (provided below).

---

# Conclusion

VLANs improve office-network organization by grouping users according to department rather than physical location. HR, IT, and Finance remain separated even when their PCs are connected to different switches. This creates smaller broadcast domains, reduces unnecessary broadcast traffic, and improves management and isolation. The 802.1Q trunks transport multiple VLANs between network devices, while Router-on-a-Stick provides controlled Layer 3 communication between the otherwise isolated VLANs. In a production network, additional access-control lists can further restrict which departments are permitted to communicate with one another.

---

## Repository contents

| File | Purpose |
| --- | --- |
| `README.md` | Project documentation, configuration guide, verification answers, troubleshooting report, and conclusion. |
| [`Proof of Concept/`](Proof%20of%20Concept/) | Submitted Packet Tracer project file and supporting configuration/topology screenshots. |
| [`Proof of Concept/Office_VLAN_Routing_Project.pkt`](Proof%20of%20Concept/Office_VLAN_Routing_Project.pkt) | Completed Cisco Packet Tracer project file. |

## Proof of Concept

The following artifacts provide visual evidence of the configured office VLAN network.

### Network topology

![Configured office VLAN topology](Proof%20of%20Concept/02-network-topology.png)

### VLAN configuration on the switch

![Switch VLAN configuration](Proof%20of%20Concept/01-switch-vlan-configuration.png)

### Finance PC IP configuration

![Finance PC static addressing](Proof%20of%20Concept/03-finance-pc-ip-configuration.png)

### Saved switch configuration

![Switch configuration saved](Proof%20of%20Concept/04-switch-configuration-saved.png)

### Router trunk interface

![Router trunk interface details](Proof%20of%20Concept/05-router-trunk-interface.png)

## License

This repository uses the [MIT License](LICENSE).
