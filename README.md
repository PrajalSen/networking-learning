# 🌐 Networking Learning

My hands-on journey toward becoming a Network Engineer — built around **Cisco Packet Tracer labs, 80/20 interview preparation, troubleshooting, and practical networking concepts**.

## 🎯 Goal

Build strong practical networking fundamentals and become job-ready through hands-on labs instead of memorizing commands.

## 🧭 Learning Map

```text
Switching
   │
   ├── VLAN
   ├── Access / Trunk
   ├── STP
   └── EtherChannel
          │
          ▼
Routing
   │
   ├── Static Routing
   ├── Inter-VLAN Routing
   └── OSPF
          │
          ▼
Network Services
   │
   ├── DHCP
   ├── DNS
   └── NAT / PAT
          │
          ▼
Security & Troubleshooting
   │
   ├── ACL
   └── Troubleshooting
```

## 🧪 Completed Labs

| Lab | Topics | Status |
|---|---|---|
| [Foundational — VLAN Basics](./labs/01-vlan-basics/) | VLAN creation, access ports, Layer 2 segmentation, verification, connectivity testing | ✅ Completed |
| [01 — VLAN + Router-on-a-Stick + DHCP + PAT](./labs/01-vlan-router-on-a-stick-dhcp-pat/) | VLAN, access ports, trunking, inter-VLAN routing, DHCP, default route, PAT, ISP loopback | ✅ Completed |

## 📅 Today's Progress — September 4, 2026

### 🟢 Topic: VLAN Basics

Today I built and documented a **Layer 2 VLAN lab** using Cisco Packet Tracer.

**Practiced:**
- Created **VLAN 10 (HR)** and **VLAN 20 (IT)**
- Configured switch ports as **access ports**
- Assigned ports to the correct VLANs
- Verified VLAN membership with `show vlan brief`
- Verified interfaces with `show interfaces status`
- Checked learned MAC addresses with `show mac address-table`
- Tested same-VLAN and cross-VLAN connectivity
- Understood why **inter-VLAN communication requires Layer 3 routing**

**80/20 takeaway:**
> **VLANs provide logical segmentation at Layer 2. Devices in different VLANs need a Layer 3 device to communicate.**

## 🔜 Upcoming

- Trunking and 802.1Q
- Router-on-a-Stick / Inter-VLAN Routing
- Static Routing vs OSPF comparison lab
- OSPF 3-router lab + failover
- Standard and Extended ACL labs
- STP root bridge / port roles lab
- EtherChannel lab
- Troubleshooting scenarios

## 🧠 My 80/20 Rule

> **Don't just learn what a command does. Learn why it is needed, what breaks if it is missing, and which `show` command proves it is working.**

Every lab therefore contains:

- 🎯 Scenario
- 🗺️ Topology
- 📋 IP addressing
- 💻 Configuration commands
- 🧠 Meaning of each important line
- 🔍 Verification commands
- 🛠️ Troubleshooting clues
- 🎤 Interview questions

## 🗺️ Visual Lab Diagram

The first completed lab combines the major concepts into one end-to-end flow:

**VLAN → Trunk → Router-on-a-Stick → DHCP → Default Route → PAT → ISP → Loopback 8.8.8.8**

![VLAN + Router-on-a-Stick + DHCP + PAT Lab](./labs/01-vlan-router-on-a-stick-dhcp-pat/topology.svg)

---

**Built while learning — one lab at a time. 🚀**
