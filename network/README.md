---
title: Network segmentation
summary: Splitting a flat home network into four VLANs on a managed switch, then trunking them into a single Proxmox node.
category: Networking
status: building
tags: [VLANs, 802.1Q, Managed switch, Proxmox]
started: 2026-03-17
publish: true
---

## VLAN Topology

| VLAN ID | Name        | Security Level | Purpose                                   |
| ------- | ----------- | -------------- | ----------------------------------------- |
| **10**  | Management  | High           | Proxmox GUI, Switch UI, Admin PC          |
| **20**  | Victim/Corp | Low            | Active Directory, Windows Targets         |
| **30**  | Attack/C2   | Restricted     | Kali Linux, Sliver C2, Exploitation Tools |
| **40**  | IoT/Lab     | Medium         | Raspberry Pi, Vulnerable Hardware         |

```mermaid
flowchart TB
    router["Home router"] -- "Port 1 · access 10" --- sw{{"Managed switch"}}
    admin["Admin PC"] -- "Port 4 · access 10" --- sw
    pi["Raspberry Pi"] -- "Port 3 · access 40" --- sw
    sw == "Port 2 · trunk 10/20/30/40" === px
    subgraph px ["Proxmox node · vmbr0 (VLAN aware)"]
        v10["VLAN 10 · Management"]
        v20["VLAN 20 · Victim/Corp"]
        v30["VLAN 30 · Attack/C2"]
    end
```

## Port Mapping & PVID

| Physical Port | Device       | Mode      | PVID | Allowed VLANs  |
| ------------- | ------------ | --------- | ---- | -------------- |
| **Port 1**    | Home Router  | Access    | 10   | 10             |
| **Port 2**    | Proxmox Node | **Trunk** | 10   | 10, 20, 30, 40 |
| **Port 3**    | Raspberry Pi | Access    | 40   | 40             |
| **Port 4**    | Admin PC     | Access    | 10   | 10             |

Configured as advanced VLAN in management portal
note: 
- tagged - 1 port to many VLANs - header to identify traffic membership
- untagged - 1 port to 1 VLAN

to show VLANs from proxmox run ```bridge vlan show``` 
