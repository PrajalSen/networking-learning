# 🧪 Lab 01 — VLAN Basics

## 🎯 Objective

Build a simple Layer 2 VLAN environment and understand how VLANs provide logical network segmentation on a switch.

This is the foundation for later labs such as Trunking, Router-on-a-Stick, Inter-VLAN Routing, and STP.

---

## 🏢 Scenario

A small company has two departments connected to the same Cisco switch:

- **HR → VLAN 10**
- **IT → VLAN 20**

The goal is to separate the departments logically using VLANs without using a router.

---

## 🌐 Topology

```text
PC1 (HR) -------- Fa0/1 ┐
                         │
PC2 (HR) -------- Fa0/2 ├── SW1
                         │
PC3 (IT) -------- Fa0/3 ┘

VLAN 10 → HR
VLAN 20 → IT
```

---

## 📋 IP Addressing

| Device | Department | VLAN | IP Address | Mask |
|---|---|---:|---|---|
| PC1 | HR | 10 | 192.168.10.10 | 255.255.255.0 |
| PC2 | HR | 10 | 192.168.10.11 | 255.255.255.0 |
| PC3 | IT | 20 | 192.168.20.10 | 255.255.255.0 |

> No default gateway is required for this Layer 2-only lab.

---

## ⚙️ Configuration

### 1. Create VLANs

```cisco
enable
configure terminal

vlan 10
 name HR
exit

vlan 20
 name IT
exit
```

### 2. Assign HR ports

```cisco
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
exit
```

### 3. Assign IT port

```cisco
interface fa0/3
 switchport mode access
 switchport access vlan 20
exit

end
write memory
```

Full configuration is available in [`configs/switch-config.txt`](./configs/switch-config.txt).

---

## 🔎 Verification

### Check VLANs and port membership

```cisco
show vlan brief
```

Expected result:

- Fa0/1 and Fa0/2 → VLAN 10 (HR)
- Fa0/3 → VLAN 20 (IT)

### Check port status

```cisco
show interfaces status
```

### Check learned MAC addresses

```cisco
show mac address-table
```

---

## 🧪 Connectivity Tests

### Test 1 — Same VLAN

From **PC1**, ping **PC2**:

```text
ping 192.168.10.11
```

✅ Expected: **Success**

Both devices belong to VLAN 10, so they can communicate at Layer 2.

### Test 2 — Different VLAN

From **PC1**, ping **PC3**:

```text
ping 192.168.20.10
```

❌ Expected: **Failure**

There is no Layer 3 device in this lab to route traffic between VLAN 10 and VLAN 20.

---

## 🛠️ Troubleshooting

If same-VLAN communication fails, check:

```cisco
show vlan brief
show interfaces status
show mac address-table
```

Common issues:

- Wrong VLAN assigned to the switch port
- Port is administratively down
- Incorrect PC IP address or subnet mask
- Cable/port mismatch in Packet Tracer

---

## 🧠 Key Takeaways

- **VLAN = logical network segmentation.**
- An **access port** normally carries traffic for one VLAN.
- Devices in the **same VLAN/subnet** can communicate without a router.
- Devices in **different VLANs** need Layer 3 routing to communicate.
- `show vlan brief` is one of the most important commands for VLAN troubleshooting.

### 80/20 Interview Rule

> **VLAN separates networks at Layer 2; routing is required to communicate between different VLANs.**

---

## 🎤 Interview Questions

1. What is a VLAN and why is it used?
2. What is the difference between an access port and a trunk port?
3. Why can PC1 communicate with PC2 but not PC3?
4. Which command verifies VLAN membership?
5. Can two different VLANs communicate without a Layer 3 device?
6. What happens to broadcast traffic inside a VLAN?
7. Why do we use VLANs instead of putting every device into one broadcast domain?

---

## ✅ Lab Status

**Completed — VLAN creation, access-port assignment, Layer 2 segmentation, verification, and connectivity testing.**
