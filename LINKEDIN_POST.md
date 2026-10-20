# LinkedIn Post — VLAN Office Network Project

🔐 **Built a secure office network using VLANs, Cisco switches, and Router-on-a-Stick routing.**

For this Cisco Packet Tracer project, I designed a small office network that separates three departments—**HR, IT, and Finance**—into independent VLANs while still enabling controlled communication between them.

### What I implemented

- VLAN 10 for HR, VLAN 20 for IT, and VLAN 30 for Finance
- Two Cisco 2960 switches with 802.1Q trunk links
- Router-on-a-Stick with one subinterface per VLAN
- Static IP addressing and default gateways for six PCs
- Same-VLAN and inter-VLAN connectivity testing
- Packet Tracer Simulation Mode analysis of ARP, VLAN tagging, frame rewriting, and routing behavior

### Key takeaway

VLANs separate users by logical function instead of physical location. That means HR, IT, and Finance can be connected across different switches while remaining in separate broadcast domains. The router only handles traffic that needs to move between VLANs, which improves network organization and isolation.

I also documented common troubleshooting checks, including verifying VLAN membership, trunk status, router subinterfaces, MAC address learning, and routing-table entries.

📂 Project documentation, Packet Tracer file, and configuration evidence:

https://github.com/gene707/Design-and-Configuration-Of-an-Office-Network-using-VLANs

#Cisco #Networking #VLAN #RouterOnAStick #PacketTracer #NetworkEngineering #CCNA #Cybersecurity
