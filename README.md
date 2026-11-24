# 🛡️ Network Access Control Lab: Extended ACLs + Port Security — Cisco Packet Tracer

A CCNA / network-security lab that protects a small network at two layers:

- **Layer 3 (Router):** an **Extended ACL** that blocks specific hosts from reaching web services (HTTP/HTTPS) on the server network.
- **Layer 2 (Switch):** **Port Security** that limits each access port to one trusted device using sticky MAC learning and shuts the port down on violation.

---

## 📌 Overview

| Layer | Device | Feature | Purpose |
|-------|--------|---------|---------|
| L3 | Router | Extended ACL (`soc`) | Deny HTTP/HTTPS from a specific client to specific servers, permit everything else |
| L2 | Switch | Port Security | Allow only 1 MAC address per access port; err-disable the port on violation |

---

## 🗺️ Network Layout

- **Client network:** `192.168.1.0/24` (PCs connected to the switch, router interface `g0/0`)
- **Server network:** `192.168.2.0/24` (servers `192.168.2.2` and `192.168.2.3`)

> Add a screenshot of the topology here: `![Topology](images/topology.png)`

---

## 🎯 Objectives

- Write a **named extended ACL** and apply it **inbound** on the client-facing interface.
- Block web traffic (TCP 80 / 443) for a single host, and for a whole subnet.
- Keep normal traffic working with an explicit `permit ip any any`.
- Enable **Port Security** with **sticky** MAC learning.
- Trigger a violation and recover the port.

---

## ⚙️ Configuration

### 1) Extended ACL — Router

**Block one host (`192.168.1.2`) from the web servers:**

```
ip access-list extended soc
 10 deny tcp host 192.168.1.2 host 192.168.2.2 eq 80
 20 deny tcp host 192.168.1.2 host 192.168.2.2 eq 443
 30 deny tcp host 192.168.1.2 host 192.168.2.3 eq 80
 40 deny tcp host 192.168.1.2 host 192.168.2.3 eq 443
 1000 permit ip any any
!
interface g0/0        ! PCs-facing port
 ip access-group soc in
```

**Block a whole network from the web servers:**

```
ip access-list extended soc
 10 deny tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 80
 20 deny tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 443
 1000 permit ip any any
!
interface g0/0
 ip access-group soc in
```

> 💡 In the destination field, use the **network address** (`192.168.2.0`) with the wildcard mask `0.0.0.255`, not a host address like `192.168.2.2`.

### 2) Port Security — Switch

```
Switch(config)# interface range f0/2-4
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport port-security
Switch(config-if-range)# switchport port-security maximum 1
Switch(config-if-range)# switchport port-security mac-address sticky
Switch(config-if-range)# switchport port-security violation shutdown
```

| Command | Meaning |
|---------|---------|
| `switchport mode access` | Port connects to an end device (required before enabling port security) |
| `maximum 1` | Only one MAC address allowed on the port |
| `mac-address sticky` | Learns the first MAC dynamically and saves it in the running config |
| `violation shutdown` | Port goes to `err-disabled` when an unknown MAC appears |

---

## ✅ Verification

**ACL**

```
show access-lists
show ip interface g0/0
```

- From the blocked PC, open the server in the browser → **request fails**.
- From another PC, open the same server → **works**.
- `show access-lists` shows the hit counters (matches) increasing on the deny lines.

**Port Security**

```
show port-security
show port-security interface f0/2
show interface status
```

- Connect a different PC to a secured port → the port becomes **err-disabled**.

**Recover the port**

```
Switch(config)# interface f0/2
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
```

---

## 📂 Repository Structure

```
.
├── ACLS_Port-Security.pkt          # Packet Tracer project file
├── ACLs_Configurations.docx        # Configuration notes
├── README.md
└── images/
    └── topology.png
```

---

## 🛠️ Tools

- Cisco Packet Tracer
- Cisco router and switch (CLI)

---

## 📚 What I Learned

- ACLs are processed **top-down** and end with an implicit `deny any`, so `permit ip any any` is needed to keep other traffic flowing.
- Extended ACLs should be placed **close to the source**; applying them **inbound** filters traffic before it is routed.
- Port Security protects against unauthorized devices plugging into the switch (MAC flooding / rogue devices).

---

## 👤 Author

**Ahmed Rady**
SOC Analyst Intern | CCNA | Ethical Hacking
🔗 [LinkedIn](https://linkedin.com/in/ahmed--rady) · 💻 [GitHub](https://github.com/AhmedRady01)
