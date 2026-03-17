## Phase 1: The Foundation (Virtualization & Compute)

To simulate complex attacks, you need multiple "victim" machines and "attacker" nodes. Don't use VirtualBox; it's too limited for high-end portfolios.

- **The Hypervisor:** Use **Proxmox VE** (Type-1 Hypervisor) or **ESXi**. This allows you to manage the entire lab via a web interface and take snapshots before "detonating" malware.
    
- **Hardware:** Aim for a dedicated machine (an old enterprise server or a high-spec NUC) with at least **64GB RAM** and **2TB NVMe SSD**. You’ll be running Windows Server, multiple Linux distros, and a SIEM simultaneously.
    
- **Networking:** Use **OPNsense** or **pfSense** as your virtual firewall. This is where you will define your zones (DMZ, Internal, Management, and "Dirty" Internet).
    

---

## Phase 2: Active Directory & The "Victim" Network

Recruiters in the Windows-dominant corporate world want to see that you understand **Active Directory (AD)** exploitation and defense.

- **The Domain:** Set up a Windows Server 2022 Domain Controller.
    
- **The Users:** Use a script (like **BadBlood**) to automatically populate your AD with thousands of fake users, groups, and permissions. This makes your "Red Teaming" look realistic.
    
- **Vulnerable Services:** Deploy "intentionally weak" machines like **Metasploitable3** or join a few Windows 10/11 workstations to the domain with common misconfigurations (e.g., LLMNR enabled, weak passwords, unpatched PrintNightmare).
    

---

## Phase 3: The "Blue Team" Stack (Visibility)

A lab without logging is just a playground. To make this "high-end," you must prove you can **see** the attacks you perform.

- **SIEM Integration:** Deploy the **Elastic Stack (ELK)** or **Wazuh**.
    
- **Endpoint Detection:** Install **Sysmon** on all Windows machines with a high-fidelity configuration (like SwiftOnSecurity’s). Forward these logs to your SIEM.
    
- **Traffic Analysis:** Set up **Zeek** or **Suricata** on your OPNsense firewall to monitor network-level threats.
    

---

## Phase 4: The "Red Team" Operations Center

This is where you demonstrate your offensive talent.

- **Command & Control (C2):** Instead of just using Metasploit, deploy a modern C2 framework like **Sliver** or **Havoc**.
    
- **The Attacker Box:** A dedicated Kali or Parrot OS instance, but customized with your own automation scripts for lateral movement and persistence.



This is the most critical phase: moving from a "flat" home network to a segmented, enterprise-grade architecture. With a **Managed Switch** and a **Proxmox Node**, you have the exact hardware required to build a "Security Onion" style defense-in-depth lab.

Here is your step-by-step blueprint to build this from the ground up.

---

## Phase 1: Physical Networking (The Trunk)

In a professional lab, the connection between your Switch and Proxmox is not just a cable; it’s a **VLAN Trunk**.

1. **Switch Config:** Access your managed switch UI. Create the following VLANs:
    
    - **VLAN 10 (Management):** For Proxmox GUI, Switch GUI, and your personal PC.
        
    - **VLAN 20 (Victim/Corp):** For Windows AD, vulnerable targets.
        
    - **VLAN 30 (Attack):** For Kali/Sliver C2.
        
    - **VLAN 40 (IoT/Lab):** Connect your **Raspberry Pi** here to simulate a vulnerable IoT device.
        
2. **Port Configuration:**
    
    - Set the port connected to your **Router** as an **Access Port** (VLAN 10).
        
    - Set the port connected to **Proxmox** as a **Trunk Port** (Tagged with 10, 20, 30, 40).
        
    - Set the port connected to the **Raspberry Pi** as an **Access Port** (VLAN 40).
        

---

## Phase 2: Proxmox Virtual Networking

You need to tell Proxmox how to handle those VLAN tags.

1. **Linux Bridge:** In Proxmox, go to `System > Network`. Edit `vmbr0`.
    
2. **VLAN Aware:** Check the **"VLAN Aware"** box. This allows VMs to be assigned to specific VLANs simply by typing the ID in their network settings.
    
3. **Management IP:** Ensure the Proxmox IP is on VLAN 10 so you don't lock yourself out.
    

---

## Phase 3: The Virtual Gateway (OPNsense)

Instead of letting your home router handle lab traffic, you will deploy a virtual firewall.

1. **Create OPNsense VM:** Give it 2 Virtual NICs.
    
    - **NIC 1 (WAN):** Connected to `vmbr0` (No VLAN tag). It gets an IP from your home router.
        
    - **NIC 2 (LAN/Trunk):** Connected to `vmbr0`. This will act as the gateway for all your internal lab VLANs.
        
2. **Firewall Rules:** This is where you prove your skill. Create "Kill-Switch" rules:
    
    - VLAN 20 (Victims) → **Deny All** to Internet.
        
    - VLAN 30 (Attack) → **Allow** to VLAN 20.
        
    - VLAN 10 (You) → **Allow** to everything (for management).
        

---

## Phase 4: Populate the Lab (Infrastructure)

Now that the "pipes" are built, add the "water."

## 1. The Windows Domain (VLAN 20)

- **VM 1:** Windows Server 2022 (Domain Controller).
    
- **VM 2:** Windows 10 Pro (Joined to the domain).
    
- **Talent Demo:** Use an **Ansible playbook** to automate the joining of the workstation to the domain. This is "Infrastructure as Code" (IaC) and recruiters love it.
    

## 2. The Defensive Watchdog (SIEM)

- **VM 3:** **Wazuh** or **Elastic Security**.
    
- **Action:** Install the Wazuh agent on your Windows VM and your Raspberry Pi. This gives you a "Single Pane of Glass" for all security events.
    

## 3. The IoT Target (The Raspberry Pi)

- **Setup:** Install a vulnerable service like a **DVWA (Damn Vulnerable Web App)** in a Docker container on the Pi.
    
- **Portfolio Hook:** "I used a Raspberry Pi to simulate a compromised IoT entry point, monitoring the lateral movement attempts via my virtual OPNsense firewall."