# FortiGate-Graduation-Project # Simulated Enterprise Branch-to-Branch Network with FortiGate & SD-WAN

[![FortiOS Version](https://img.shields.io/badge/FortiOS-v7.6.7-orange.svg)](https://www.fortinet.com)
[![Platform](https://img.shields.io/badge/Platform-EVE--NG%20Community-blue.svg)](https://www.eve-ng.net/)
[![Project Status](https://img.shields.io/badge/Status-Completed-success.svg)]()

## 📌 Project Overview
This graduation project demonstrates the implementation of a simulated enterprise network connecting two virtual company branche. The architecture utilizes two **FortiGate-VM64-KVM** firewalls running **FortiOS 7.6.7** to enforce security policies, manage routing, secure inter-branch communications, and optimize WAN traffic with SD-WAN redundancy.

The project was built and tested inside the **EVE-NG Community Edition** environment as part of the **FortiOS 7.6 Administrator** track at **Creativa**[cite: 1].

---

## 🛠 Project Scope & Requirements

| # | Task | Status | Purpose |
|---|---|---|---|
| 1 | **Account Configuration** | Completed | Administrative access control with custom privileges. |
| 2 | **Interface Configuration** | Completed | Assigning static IP addressing to all branch interfaces. |
| 3 | **Routing** | Completed | Static route creation for inter-branch reachability. |
| 4 | **Firewall Policy** | Completed | Granular traffic control and NAT translation. |
| 5 | **Authentication** | Completed | User authentication for outbound web access. |
| 6 | **Security Profiles** | Completed | Deep inspection via Antivirus & SSL/SSH inspection. |
| 7 | **Site-to-Site VPN** | Completed | IPsec encrypted tunnel for branch-to-branch communication. |
| 8 | **SD-WAN (Bonus)** | Completed | Dual-WAN link load balancing and failover mechanism. |

---

## 📐 Network Topology & Addressing Table

### Network Diagram
```text
                        [ WAN-Link2 (Cloud) ]
                        /                    \
                    port4                    port4
                      /                        \
   [VPC3]  [VPC4]  Fortinet1 --- port1==WAN1==port1 --- Fortinet2  [VPC5]  [VPC6]
      \      /       |  \                              /  |        \      /
    eth0    eth0   port2 port3                     port2 port3    eth0    eth0
```

### IP Addressing Table

| Device | Interface | IP Address / Subnet | Connected Subnet / Role |
|---|---|---|---|
| **Fortinet1** | `port1` | `10.0.0.1/24` | Primary WAN Link (WAN1)[cite: 1] |
| **Fortinet1** | `port2` | `192.168.10.1/24` | Internal LAN 1[cite: 1] |
| **Fortinet1** | `port3` | `192.168.20.1/24` | Internal LAN 2[cite: 1] |
| **Fortinet1** | `port4` | `20.0.0.1/24` | Secondary WAN Link (SD-WAN WAN2)[cite: 1] |
| **Fortinet2** | `port1` | `10.0.0.2/24` | Primary WAN Link (WAN1)[cite: 1] |
| **Fortinet2** | `port2` | `192.168.30.1/24` | Internal LAN 1[cite: 1] |
| **Fortinet2** | `port3` | `192.168.40.1/24` | Internal LAN 2[cite: 1] |
| **Fortinet2** | `port4` | `20.0.0.2/24` | Secondary WAN Link (SD-WAN WAN2)[cite: 1] |
| **VPC3** | `eth0` | `192.168.10.10/24` | End Device (Branch 1)[cite: 1] |
| **VPC4** | `eth0` | `192.168.20.10/24` | End Device (Branch 1)[cite: 1] |
| **VPC5** | `eth0` | `192.168.30.10/24` | End Device (Branch 2)[cite: 1] |
| **VPC6** | `eth0` | `192.168.40.10/24` | End Device (Branch 2)[cite: 1] |

---

## ⚙️ Configuration Summary

### 1. Account Management
Created a dedicated administrative user with `super_admin` privileges on both firewalls[cite: 1]:
```custom-cli
config system admin
    edit "nadia-admin"
        set password <strong-password>
        set accprofile "super_admin"
    end
```

### 2. Interface Configuration
Configured static IP addresses and enabled administrative access services[cite: 1]:
```custom-cli
config system interface
    edit "port1"
        set mode static
        set ip 10.0.0.1 255.255.255.0
        set allowaccess ping https ssh
    next
    edit "port2"
        set mode static
        set ip 192.168.10.1 255.255.255.0
        set allowaccess ping https ssh
    end
```

### 3. Static Routing & Firewall Policies
Set up routing tables and bidirectional policies with NAT enabled[cite: 1]:
```custom-cli
config router static
    edit 1
        set dst 192.168.30.0 255.255.255.0
        set gateway 10.0.0.2
        set device "port1"
    end

config firewall policy
    edit 1
        set name "Internal-to-WAN"
        set srcintf "port2" "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    end
```

### 4. Identity Authentication & Security Inspection
Enforced local user group authentication and UTM security profile inspection[cite: 1]:
```custom-cli
config firewall policy
    edit 1
        set groups "AuthGroup"
        set utm-status enable
        set av-profile "default"
        set webfilter-profile "default"
        set ssl-ssh-profile "certificate-inspection"
    end
```

### 5. IPsec Site-to-Site VPN
Encrypted all inter-branch communication across public networks[cite: 1]:
```custom-cli
config vpn ipsec phase1-interface
    edit "VPN-F1-F2"
        set interface "port1"
        set peertype any
        set proposal des-sha256
        set remote-gw 10.0.0.2
        set psksecret <shared-secret>
    end
```

### 6. SD-WAN Implementation (Bonus Task)
Configured dual WAN links under a `virtual-wan-link` zone for dynamic traffic management and failover[cite: 1]:
```custom-cli
config system sdwan
    set status enable
    config zone
        edit "virtual-wan-link"
        next
    end
    config members
        edit 1
            set interface "port4"
            set gateway 20.0.0.2
        next
    end
end
```

---

## 🧪 Verification & Testing

### 1. IPsec Tunnel Status Check
```custom-cli
FortiGate-VM64-KVM # diagnose vpn ike gateway list
name: VPN-F1-F2
addr: 10.0.0.1:500 -> 10.0.0.2:500
IKE SA: created 1/1  established 1/1  time 9040/9040/9040 ms
status: established
```

### 2. SD-WAN Member Status Check
```custom-cli
FortiGate-VM64-KVM # diagnose sys sdwan member
Member(1) : interface: port4, gateway: 20.0.0.2, priority: 1, weight: 0
```

### 3. End-to-End ICMP Connectivity Tests
Connectivity tests confirmed that traffic successfully travels across the encrypted VPN and SD-WAN path between branches:

| Source Device | Destination IP | Result |
|---|---|---|
| **VPC3** | `192.168.30.10` (VPC5) | **Success** (TTL=62, Time <= 1.2ms) |
| **VPC4** | `192.168.40.10` (VPC6) | **Success** |

---

## 🎓 Project Details
- **Organization:** Creativa
- **Environment:** EVE-NG Community Edition
- **Device Version:** FortiGate-VM64-KVM v7.6.7
- **Project Duration:** August 11 – October 25, 2026
