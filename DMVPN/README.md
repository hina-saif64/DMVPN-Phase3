# DMVPN Phase 3 Lab

A Cisco IOS lab demonstrating a **Dynamic Multipoint VPN (DMVPN) Phase 3** topology with a central hub, two spokes, an ISP transit router, and three end-user LANs.

## 📌 Overview

This lab is built around a hub-and-spoke DMVPN design:

- **Hub** — central DMVPN hub and EIGRP aggregation point
- **Spoke1** — first DMVPN spoke
- **Spoke2** — second DMVPN spoke
- **ISP** — transit router providing WAN connectivity
- **User1 / User2 / User3** — endpoint hosts connected to the respective LANs

The topology uses:

- GRE multipoint tunnels
- NHRP
- IPsec protection
- EIGRP AS 100
- Static default routes toward the ISP
- Separate WAN and LAN addressing

## 🗺️ Topology

![DMVPN Phase 3 Topology](topology/DMVPN-Lab.png)

### Logical layout

```text
                         +-----------+
                         |    ISP    |
                         +-----------+
                         /     |                             /      |                             /       |                          +------+ +------+ +------+
                   | Hub  | |Spoke1| |Spoke2|
                   +------+ +------+ +------+
                      |         |        |
                   Switch1    Switch2  Switch3
                      |         |        |
                   User1      User2     User3
```

The Hub provides the DMVPN/NHRP service for the spokes. The spokes establish their WAN connectivity through the ISP router and use `Tunnel1` for the DMVPN overlay.

## 🌐 IP Addressing

| Device | Interface | Address | Purpose |
|---|---|---|---|
| Hub | Gi0/0 | `11.1.1.1/22` | WAN |
| Hub | Gi0/1 | `10.1.1.100/24` | LAN |
| Hub | Tunnel1 | `192.168.1.1/24` | DMVPN |
| Spoke1 | Gi0/0 | `22.1.1.1/22` | WAN |
| Spoke1 | Gi0/1 | `10.1.2.100/24` | LAN |
| Spoke1 | Tunnel1 | `192.168.1.2/24` | DMVPN |
| Spoke2 | Gi0/0 | `33.1.1.1/22` | WAN |
| Spoke2 | Gi0/1 | `10.1.3.100/24` | LAN |
| Spoke2 | Tunnel1 | `192.168.1.3/24` | DMVPN |
| ISP | Gi0/0 | `11.1.1.2/22` | Hub WAN |
| ISP | Gi0/1 | `22.1.1.2/22` | Spoke1 WAN |
| ISP | Gi0/2 | `33.1.1.2/22` | Spoke2 WAN |

### Endpoints

| Host | Address |
|---|---|
| User1 | `10.1.1.1/24` |
| User2 | `10.1.2.1/24` |
| User3 | `10.1.3.1/24` |

## 🔐 DMVPN Configuration

The DMVPN overlay uses `Tunnel1` on the Hub and both spokes.

### Hub

The Hub uses:

- Tunnel address: `192.168.1.1/24`
- NHRP authentication: `cisco123`
- NHRP network ID: `1`
- GRE multipoint tunnel mode
- IPsec profile: `prof`
- WAN tunnel source: `11.1.1.1`

The Hub also defines multicast NHRP mappings for both spokes' WAN addresses.

### Spoke1

Spoke1 uses:

- Tunnel address: `192.168.1.2/24`
- NHRP NHS: `192.168.1.1`
- Hub WAN address: `11.1.1.1`
- WAN tunnel source: `22.1.1.1`
- GRE multipoint tunnel mode
- IPsec profile: `prof`

### Spoke2

Spoke2 uses:

- Tunnel address: `192.168.1.3/24`
- NHRP NHS: `192.168.1.1`
- Hub WAN address: `11.1.1.1`
- WAN tunnel source: `33.1.1.1`
- GRE multipoint tunnel mode
- IPsec profile: `prof`

## 🔒 IPsec

The lab protects the GRE tunnel using IPsec.

The configured ISAKMP policy uses:

```text
Encryption:      AES
Hash:            SHA-256
Authentication:  Pre-shared key
DH Group:        5
```

The pre-shared key configured in the supplied router configurations is:

```text
cisco123
```

The IPsec transform set is:

```text
TSET
esp-aes
esp-sha-hmac
mode tunnel
```

The `prof` crypto IPsec profile applies this transform set and uses a 900-second security-association lifetime.

> **Security note:** This repository is intended as a lab/learning project. The supplied configuration contains a lab pre-shared key. Do not reuse this credential in production.

## 🧭 Routing

The routers use **EIGRP autonomous system 100**.

The Hub, Spoke1, and Spoke2 configurations include:

```text
router eigrp 100
 network 10.0.0.0
 network 192.168.1.0
```

The Hub additionally summarizes the EIGRP routes on the DMVPN tunnel:

```text
ip summary-address eigrp 100 10.1.0.0 255.255.0.0
```

Each DMVPN router also has a default route toward the ISP:

```text
Hub:     0.0.0.0/0 -> 11.1.1.2
Spoke1:  0.0.0.0/0 -> 22.1.1.2
Spoke2:  0.0.0.0/0 -> 33.1.1.2
```

## 🧪 Lab Objectives

This lab can be used to practice and demonstrate:

- DMVPN Phase 3 topology design
- Hub-and-spoke DMVPN architecture
- GRE multipoint tunnels
- NHRP registration and resolution
- NHRP NHS configuration
- Dynamic spoke-to-spoke communication
- IPsec protection of GRE tunnels
- EIGRP over a DMVPN overlay
- EIGRP route summarization
- WAN/LAN addressing
- Cisco IOS troubleshooting

## 🔍 Useful Verification Commands

After configuring the topology, useful Cisco IOS verification commands include:

```text
show ip interface brief
show interfaces Tunnel1
show dmvpn
show ip nhrp
show ip nhrp brief
show ip route
show ip route eigrp
show ip eigrp neighbors
show ip eigrp topology
show crypto isakmp sa
show crypto ipsec sa
```

Connectivity can also be tested with:

```text
ping 192.168.1.1
ping 192.168.1.2
ping 192.168.1.3
```

And from the LANs, test reachability between the configured endpoint networks.

## 📁 Repository Structure

```text
DMVPN-Phase3-Lab/
│
├── README.md
│
├── topology/
│   └── DMVPN-Lab.png
│
└── configs/
    ├── Hub.txt
    ├── Spoke1.txt
    ├── Spoke2.txt
    └── ISP.txt
```

## 📄 Configuration Files

The repository contains the running configurations used for each router:

- [`Hub.txt`](configs/Hub.txt) — DMVPN Hub configuration
- [`Spoke1.txt`](configs/Spoke1.txt) — DMVPN Spoke 1 configuration
- [`Spoke2.txt`](configs/Spoke2.txt) — DMVPN Spoke 2 configuration
- [`ISP.txt`](configs/ISP.txt) — ISP/transit router configuration

## 🛠️ Technologies

| Technology | Role |
|---|---|
| DMVPN | Dynamic VPN overlay |
| mGRE | Multipoint GRE tunnel transport |
| NHRP | Dynamic next-hop/peer resolution |
| IPsec | Tunnel protection |
| EIGRP | Dynamic routing |
| Cisco IOS | Router operating system |

## 📚 Learning Notes

This project is intended for networking practice and can be extended with additional exercises such as:

1. Verify NHRP registration from both spokes.
2. Verify the Hub's NHRP mappings.
3. Verify EIGRP neighbor relationships.
4. Confirm LAN routes are learned through EIGRP.
5. Test connectivity between User1, User2, and User3.
6. Inspect the IPsec security associations.
7. Test and troubleshoot DMVPN spoke-to-spoke traffic.
8. Capture and analyze the control-plane and data-plane behavior.

## ⚠️ Lab Disclaimer

This configuration is provided for educational and lab use. Interface names, IP addressing, authentication credentials, routing design, and security parameters should be reviewed and adapted before being used in any production environment.

---

**Project:** DMVPN Phase 3 Lab  
**Platform:** Cisco IOS  
**Routing:** EIGRP AS 100  
**VPN:** DMVPN + IPsec  
