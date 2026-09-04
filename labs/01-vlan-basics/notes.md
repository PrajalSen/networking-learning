# 📝 VLAN Basics — Study Notes

## What I Learned Today

### 1. VLANs create logical separation

A VLAN divides one physical switch into multiple logical networks.

In this lab:

- **VLAN 10 → HR**
- **VLAN 20 → IT**

Even though both departments use the same physical switch, they are separated at Layer 2.

---

## 2. Access Ports

An access port is normally assigned to a single VLAN and is used to connect end devices such as PCs.

Example:

```cisco
interface fa0/1
 switchport mode access
 switchport access vlan 10
```

This places Fa0/1 into VLAN 10.

---

## 3. Same VLAN vs Different VLAN

### Same VLAN

PC1 → VLAN 10  
PC2 → VLAN 10

```text
PC1 ───── Switch ───── PC2
        VLAN 10
```

They can communicate at Layer 2.

### Different VLAN

PC1 → VLAN 10  
PC3 → VLAN 20

```text
PC1 ───── Switch ───── PC3
       VLAN 10 | VLAN 20
```

They cannot communicate directly because VLANs are separate broadcast domains.

A Layer 3 device such as a router or Layer 3 switch is required for inter-VLAN communication.

---

## 4. Important Commands

### Check VLANs and port membership

```cisco
show vlan brief
```

**First command to check when a VLAN connectivity problem appears.**

### Check interface status

```cisco
show interfaces status
```

Useful for checking whether ports are connected and operational.

### Check learned MAC addresses

```cisco
show mac address-table
```

Useful for verifying that the switch is learning Layer 2 addresses.

---

## 🔧 Troubleshooting Thought Process

If **PC1 cannot ping PC2** and both should be in VLAN 10:

1. Check PC IP address and subnet mask.
2. Check the switch port status.
3. Check VLAN membership:

```cisco
show vlan brief
```

4. Confirm both ports are assigned to VLAN 10.
5. Check whether the switch has learned the PCs' MAC addresses:

```cisco
show mac address-table
```

6. Check the cable and Packet Tracer port selection.

### Interview mindset

Don't randomly change configuration. **Verify Layer 1 → Layer 2 → Layer 3 in order.**

For this lab, Layer 3 routing does not exist, so the main focus is Layer 1/Layer 2.

---

## 🎯 80/20 Interview Knowledge

Remember these points:

- **VLAN = logical segmentation at Layer 2.**
- Each VLAN is a separate **broadcast domain**.
- **Access port = one VLAN for an end device.**
- Same VLAN → Layer 2 communication is possible.
- Different VLANs → Layer 3 routing is required.
- `show vlan brief` → verify VLAN membership.
- `show interfaces status` → verify port status.
- `show mac address-table` → verify learned MAC addresses.

### One-line interview answer

> **A VLAN logically separates devices into different Layer 2 broadcast domains on the same physical switching infrastructure. Communication between different VLANs requires Layer 3 routing.**

---

## 🧪 Lab Validation

| Test | Expected Result |
|---|---|
| PC1 → PC2 | ✅ Ping succeeds |
| PC3 → PC4 | ✅ Ping succeeds |
| PC1 → PC3 | ❌ Ping fails |
| VLAN 10 → VLAN 20 without router | ❌ No communication |

---

## 🚀 Next Concept

**Trunking and 802.1Q** — understand how multiple VLANs can travel across a single switch-to-switch or switch-to-router link.
