# 🧪 Lab 01 — VLAN + Router-on-a-Stick + DHCP + PAT

## 🎯 Scenario

A small company has two departments:

- **VLAN 10 — HR**
- **VLAN 20 — IT**

Both departments use the same switch, but they must remain logically separated. A router provides inter-VLAN routing, DHCP, and PAT for outside connectivity. The ISP router is represented by a Cisco router with a loopback address acting as a fake Internet destination.

## 🗺️ Topology

```text
 PC1 / HR                         PC2 / HR
192.168.10.x                    192.168.10.x
      │                               │
      └──────────┐       ┌────────────┘
                 ▼       ▼
              ┌─────────────┐
              │   SWITCH    │
              │             │
              │ Fa0/1 VLAN10│
              │ Fa0/2 VLAN10│
              │ Fa0/4 VLAN20│
              │ Fa0/3 TRUNK │
              └──────┬──────┘
                     │
                  802.1Q
                   TRUNK
                     │
                     ▼
              ┌─────────────┐
              │     R1      │
              │             │
              │ G0/0.10    │── 192.168.10.1 (HR GW)
              │ G0/0.20    │── 192.168.20.1 (IT GW)
              │             │
              │ DHCP        │
              │ PAT         │
              └──────┬──────┘
                     │ G0/1
                  200.1.1.1
                     │
                     ▼
              ┌─────────────┐
              │     ISP     │
              │ 200.1.1.2   │
              │             │
              │ Lo0         │
              │ 8.8.8.8     │
              └─────────────┘

PC3 / IT
192.168.20.x
      │
      └──── Fa0/4 → VLAN 20
```

> **Note:** In Packet Tracer, the ISP side can simply be another router. Its Loopback0 `8.8.8.8/32` is a **fake Internet destination** for lab testing.

---

## 📋 IP Addressing Plan

| Device / Interface | Address | Purpose |
|---|---|---|
| PC1 | DHCP — 192.168.10.x | HR client |
| PC2 | DHCP — 192.168.10.x | HR client |
| PC3 | DHCP — 192.168.20.x | IT client |
| R1 G0/0.10 | 192.168.10.1/24 | VLAN 10 gateway |
| R1 G0/0.20 | 192.168.20.1/24 | VLAN 20 gateway |
| R1 G0/1 | 200.1.1.1/24 | Outside/PAT interface |
| ISP G0/0 | 200.1.1.2/24 | ISP next hop |
| ISP Loopback0 | 8.8.8.8/32 | Fake Internet destination |

---

# STEP 1 — Create VLANs

On the switch:

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

### 🧠 Meaning

- `vlan 10` → creates VLAN 10.
- `name HR` → gives VLAN 10 a readable name.
- `vlan 20` → creates VLAN 20.
- `name IT` → gives VLAN 20 a readable name.

### 🔍 Verify

```cisco
show vlan brief
```

Expected idea:

```text
10  HR   active
20  IT   active
```

**Why verify?** Make sure the VLANs actually exist before assigning ports.

---

# STEP 2 — Assign Access Ports

Example used in this lab:

```cisco
interface fa0/1
switchport mode access
switchport access vlan 10
exit

interface fa0/2
switchport mode access
switchport access vlan 10
exit

interface fa0/4
switchport mode access
switchport access vlan 20
exit
```

### 🧠 Meaning

```text
switchport mode access
```
→ Makes the port an **access port** for an end device.

```text
switchport access vlan 10
```
→ Places that port into VLAN 10.

So:

```text
Fa0/1 → VLAN 10 → HR
Fa0/2 → VLAN 10 → HR
Fa0/4 → VLAN 20 → IT
```

### 🔍 Verify

```cisco
show vlan brief
```

Confirm the ports appear under the correct VLAN.

---

# STEP 3 — Configure the Switch-to-Router Trunk

The switch port connected to R1 is **Fa0/3**.

```cisco
interface fa0/3
switchport mode trunk
exit
```

### 🧠 Meaning

```text
switchport mode trunk
```
→ Allows the link to carry **multiple VLANs**.

```text
VLAN 10 ─┐
         ├── Fa0/3 TRUNK ── R1
VLAN 20 ─┘
```

### 🔍 Verify

```cisco
show interfaces trunk
```

Look for:

```text
Fa0/3    ...    trunking
```

and confirm VLAN 10 and VLAN 20 are active/allowed.

---

# STEP 4 — Configure Router-on-a-Stick

On R1:

```cisco
enable
configure terminal

interface g0/0
no shutdown
exit

interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
```

### 🧠 Meaning

```text
interface g0/0.10
```
→ Creates a virtual subinterface for VLAN 10.

```text
encapsulation dot1Q 10
```
→ Associates this subinterface with VLAN 10 using 802.1Q tagging.

```text
ip address 192.168.10.1 ...
```
→ Makes `192.168.10.1` the gateway for VLAN 10.

Same idea for VLAN 20:

```text
G0/0.20 → VLAN 20 → 192.168.20.1
```

### 🔍 Verify

```cisco
show ip interface brief
```

You want:

```text
G0/0.10   192.168.10.1   up   up
G0/0.20   192.168.20.1   up   up
```

> **Important troubleshooting note:** If you see something like `G0/1 200.1.1.1 up down`, the physical interface is enabled, but the other end (ISP router) isn't connected/configured yet. We'll fix that next.

---

# STEP 5 — Test Inter-VLAN Routing

From a VLAN 10 PC:

```text
ping 192.168.10.1
ping 192.168.20.1
```

Then test the VLAN 20 client:

```text
ping 192.168.20.2
```

### 🧠 What this proves

```text
PC VLAN 10
    ↓
Switch
    ↓ trunk
R1 G0/0.10
    ↓
R1 G0/0.20
    ↓
PC VLAN 20
```

If the VLAN 10 PC can reach the VLAN 20 PC, **inter-VLAN routing works**.

---

# STEP 6 — Configure DHCP on R1

## VLAN 10

```cisco
ip dhcp excluded-address 192.168.10.1

ip dhcp pool VLAN10
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit
```

## VLAN 20

```cisco
ip dhcp excluded-address 192.168.20.1

ip dhcp pool VLAN20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit
```

## 🧠 What each line means

```cisco
ip dhcp excluded-address 192.168.10.1
```
→ **Don't give `.1` to a PC because it is the router's gateway.**

```cisco
ip dhcp pool VLAN10
```
→ Creates a DHCP pool named `VLAN10`.

```cisco
network 192.168.10.0 255.255.255.0
```
→ Defines the network from which DHCP addresses are allocated.

```cisco
default-router 192.168.10.1
```
→ Gives clients the router's IP as their **default gateway**.

```cisco
dns-server 8.8.8.8
```
→ Gives clients a DNS server so domain names can be resolved.

### 🔍 Verify

On a PC select **Desktop → IP Configuration → DHCP**.

Then on R1:

```cisco
show ip dhcp binding
```

Expected example:

```text
192.168.10.2   Automatic
192.168.10.3   Automatic
192.168.20.2   Automatic
```

---

# STEP 7 — Configure the ISP Router

On the ISP router:

```cisco
enable
configure terminal

interface g0/0
ip address 200.1.1.2 255.255.255.0
no shutdown
exit
```

On R1, G0/1 is:

```text
200.1.1.1/24
```

Test from R1:

```cisco
ping 200.1.1.2
```

Expected:

```text
!!!!!
Success rate is 100 percent
```

---

# STEP 8 — Configure PAT

## 8.1 Select private networks

On R1:

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
```

### 🧠 Meaning

This ACL tells NAT:

> These two private networks are allowed to be translated.

---

## 8.2 Mark inside interfaces

```cisco
interface g0/0.10
ip nat inside
exit

interface g0/0.20
ip nat inside
exit
```

→ Traffic entering from the VLANs is considered **inside** NAT traffic.

## 8.3 Mark outside interface

```cisco
interface g0/1
ip nat outside
exit
```

→ G0/1 faces the ISP, so it is the **outside** interface.

## 8.4 Enable PAT

```cisco
ip nat inside source list 1 interface g0/1 overload
```

### 🧠 Break it down

```text
inside source → translate inside/private addresses
list 1        → use ACL 1
interface g0/1 → use G0/1's public IP
              → 200.1.1.1
overload      → many private clients share one public IP using ports
```

### 🔍 Verify

Before traffic, it is normal to see:

```text
Total translations: 0
Hits: 0
```

After generating traffic, check:

```cisco
show ip nat translations
show ip nat statistics
```

Example:

```text
Inside local       Inside global
192.168.10.2:13 → 200.1.1.1:13
192.168.10.2:14 → 200.1.1.1:14
```

This proves PAT is translating the private client to the public IP.

---

# STEP 9 — Configure the Default Route

On R1:

```cisco
ip route 0.0.0.0 0.0.0.0 200.1.1.2
```

### 🧠 Meaning

> For any destination R1 does not already know, send the packet to the ISP at `200.1.1.2`.

Think:

```text
Unknown destination → ISP
```

### 🔍 Verify

```cisco
show ip route
```

Look for the default route:

```text
S* 0.0.0.0/0 via 200.1.1.2
```

---

# STEP 10 — Create the Fake Internet with Loopback

On the ISP router:

```cisco
interface loopback 0
ip address 8.8.8.8 255.255.255.255
exit
```

### 🧠 What is Loopback?

A Loopback interface is a **virtual interface** on the router. It stays up independently of a physical cable and is useful for lab/testing purposes.

Here we use:

```text
8.8.8.8/32
```

as our **fake Internet destination**.

---

# STEP 11 — End-to-End Test 🚀

From PC1:

```text
ping 8.8.8.8
```

Expected:

```text
Reply from 8.8.8.8
0% loss
```

The complete journey is:

```text
192.168.10.2
     │
     ▼
 VLAN 10
     │
     ▼
 Switch
     │ 802.1Q trunk
     ▼
 R1 G0/0.10
     │
     │ PAT
     ▼
200.1.1.1
     │
     ▼
200.1.1.2 ISP
     │
     ▼
8.8.8.8 Loopback
```

---

# 🔍 Verification Checklist

Run these after each major stage:

| Stage | Verify | What you're checking |
|---|---|---|
| VLAN | `show vlan brief` | VLANs + access ports |
| Trunk | `show interfaces trunk` | Fa0/3 is trunking |
| Router interfaces | `show ip interface brief` | IPs and up/up status |
| DHCP | `show ip dhcp binding` | Client leases |
| Routing | `show ip route` | Connected/default routes |
| NAT | `show ip nat statistics` | Inside/outside mapping |
| PAT | `show ip nat translations` | Actual translations |
| Connectivity | `ping` | End-to-end reachability |

---

# 🛠️ Quick Troubleshooting

### PC cannot get DHCP address

Check:

```cisco
show ip dhcp binding
show ip interface brief
```

Then check VLAN assignment:

```cisco
show vlan brief
```

### VLAN 10 cannot reach VLAN 20

Check:

```cisco
show interfaces trunk
show ip interface brief
```

Confirm:

```text
G0/0.10 → VLAN 10
G0/0.20 → VLAN 20
```

### PC can reach R1 but not 8.8.8.8

Check:

```cisco
show ip route
show ip nat translations
show ip nat statistics
```

Also verify:

```text
R1 G0/1       = 200.1.1.1
ISP G0/0      = 200.1.1.2
R1 default route → 200.1.1.2
```

### `up/down` appears on an interface

Example:

```text
G0/1  200.1.1.1  up  down
```

The physical interface is enabled, but the **other end is not connected/configured correctly**. Check the ISP-side interface and cable.

---

# 🎤 Interview Questions

1. What is the difference between an access port and a trunk port?
2. Why is trunking required in Router-on-a-Stick?
3. Why do we create subinterfaces such as `G0/0.10`?
4. What does `encapsulation dot1Q 10` do?
5. Why is `192.168.10.1` the default gateway for VLAN 10?
6. What does `ip dhcp excluded-address` do?
7. What is the difference between NAT and PAT?
8. What does `overload` mean in the PAT command?
9. Why do we need a default route?
10. What is a Loopback interface and why did we use one here?
11. Which commands would you use to troubleshoot DHCP?
12. Which command proves that PAT is actually translating traffic?

---

# 🧠 80/20 Takeaway

Remember the whole lab as:

```text
VLAN separates
      ↓
Access ports assign devices
      ↓
Trunk carries multiple VLANs
      ↓
Router-on-a-Stick routes between VLANs
      ↓
DHCP assigns IP configuration
      ↓
Default route points unknown traffic to ISP
      ↓
PAT translates private → public
      ↓
ISP → Loopback (fake Internet)
```

> **Golden rule:** Configure → Verify → Test → Troubleshoot.

This lab connects several networking concepts into one practical scenario instead of treating them as isolated commands.
