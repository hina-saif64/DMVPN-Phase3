# Cisco IPSec Site-to-Site VPN Lab

A Cisco IOS networking lab demonstrating the configuration, operation, verification, and troubleshooting of an **IPSec site-to-site VPN using IKEv1**.

The lab connects two remote LANs through an ISP router, with R1 and R2 establishing an encrypted VPN tunnel between them.

---

## 📌 Project Overview

This project demonstrates how two Cisco routers can securely communicate across an untrusted transit network using **IPSec VPN**.

### Network Flow

```text
LAN 1
10.1.1.0/24
     |
     |
    R1
11.1.1.1/30
     |
     |
   ISP
     |
     |
22.1.1.1/30
    R2
     |
     |
LAN 2
10.1.2.0/24
```

The ISP router represents the Internet/transit network between the two VPN endpoints.

Traffic between:

```text
10.1.1.0/24 <---- IPSec VPN ----> 10.1.2.0/24
```

is encrypted using IPSec.

---

## 🎯 Objectives

This lab demonstrates:

- Cisco IOS IPSec VPN configuration
- IKEv1 Phase 1 negotiation
- IPSec Phase 2 negotiation
- Pre-shared-key authentication
- AES encryption
- SHA-based integrity protection
- Diffie-Hellman key exchange
- Interesting traffic using extended ACLs
- Crypto map configuration
- VPN peer configuration
- Routing through an ISP/transit router
- VPN verification and troubleshooting

---

## 🗺️ Addressing

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | E0/0 | 11.1.1.1/30 | WAN / VPN |
| R1 | E0/1 | 10.1.1.100/24 | Branch 1 LAN |
| ISP | E0/0 | 11.1.1.2/30 | Transit to R1 |
| ISP | E0/1 | 22.1.1.2/30 | Transit to R2 |
| ISP | Lo1 | 8.8.8.8/24 | Simulated Internet |
| R2 | E0/0 | 22.1.1.1/30 | WAN / VPN |
| R2 | E0/1 | 10.1.2.100/24 | Branch 2 LAN |

---

# 🔐 VPN Configuration

## IKEv1 Phase 1

R1 and R2 use matching IKEv1 parameters to establish the secure management channel used to negotiate the VPN.

| Parameter | Configuration |
|---|---|
| Encryption | AES |
| Hash | SHA-256 |
| Authentication | Pre-shared key |
| Diffie-Hellman | Group 5 |
| Pre-shared Key | `cisco123` |

### Phase 1 Purpose

IKE Phase 1 establishes a secure channel between the VPN peers.

The peers negotiate:

1. IKE security parameters
2. Diffie-Hellman key material
3. Peer authentication
4. Secure communication for the next phase

---

# 🔒 IPSec Phase 2

Once IKE Phase 1 has completed, IPSec Phase 2 negotiates the parameters used to protect the actual user traffic.

### IPSec Parameters

| Parameter | Configuration |
|---|---|
| Transform Set | `TRANS` |
| Encryption | ESP-AES |
| Integrity | ESP-SHA-HMAC |
| Mode | Tunnel |

The resulting IPSec Security Associations protect traffic between the two branch LANs.

---

# 🎯 Interesting Traffic

The VPN does not encrypt every packet automatically.

The crypto ACL identifies the traffic that should be protected.

### R1

```text
10.1.1.0/24 → 10.1.2.0/24
```

### R2

```text
10.1.2.0/24 → 10.1.1.0/24
```

The ACLs are mirror images of each other.

This allows both routers to identify the traffic that belongs inside the VPN tunnel.

---

# 🔑 Crypto Map

The crypto map connects the main IPSec components:

```text
Remote Peer
     |
     +---- Transform Set
     |
     +---- Crypto ACL
```

The crypto map is applied to the WAN-facing interface on both R1 and R2.

When interesting traffic is detected, the router can initiate or use the IPSec tunnel.

---

# 🌐 Routing

R1 uses the ISP router as its default gateway:

```text
R1
0.0.0.0/0 → 11.1.1.2
```

R2 uses:

```text
R2
0.0.0.0/0 → 22.1.1.2
```

The ISP provides the transit path between the two VPN endpoints.

---

# 🔄 IKEv1 Message Exchange

## Main Mode

IKEv1 Main Mode uses six messages.

```text
R1                                      R2
 |                                       |
 |  1. IKE SA Proposal ----------------> |
 |  2. IKE SA Response <--------------- |
 |                                       |
 |  3. DH Exchange --------------------> |
 |  4. DH Exchange <------------------- |
 |                                       |
 |  5. Authentication ----------------> |
 |  6. Authentication <--------------- |
 |                                       |
 |        IKE Phase 1 Complete           |
```

### Main Mode

The exchange establishes:

- IKE security parameters
- Diffie-Hellman key material
- Peer authentication

---

# 🔄 IPSec Quick Mode

After Phase 1, Quick Mode is used for IPSec Phase 2.

```text
R1                                      R2
 |                                       |
 |  1. IPSec Proposal ----------------> |
 |  2. IPSec Response <--------------- |
 |  3. Confirmation ------------------> |
 |                                       |
 |       IPSec SA Established            |
```

The resulting Security Associations protect the actual VPN traffic.

---

# 🚦 Complete VPN Flow

```text
Client/Host
    |
    v
Interesting Traffic
    |
    v
Crypto ACL Match
    |
    v
IKE Phase 1
    |
    v
IKE Authentication
    |
    v
IPSec Phase 2
    |
    v
ESP Encryption
    |
    v
ISP / Transit Network
    |
    v
ESP Decryption
    |
    v
Remote LAN
```

---

# 🧪 Verification

After generating traffic between the two LANs, the following commands can be used to verify the VPN.

### Check IKE

```cisco
show crypto isakmp sa
```

### Check IPSec

```cisco
show crypto ipsec sa
```

### Check Crypto Map

```cisco
show crypto map
```

### Check ACLs

```cisco
show access-lists
```

### Check Routing

```cisco
show ip route
```

### Check Interfaces

```cisco
show interfaces
```

### Test Connectivity

```cisco
ping <remote-host>
```

---

# 🛠️ Troubleshooting

If the VPN does not establish, check the following:

### 1. Interface Status

```cisco
show ip interface brief
```

Verify that the WAN interfaces are operational.

### 2. Peer Connectivity

Confirm that R1 can reach R2's WAN address and vice versa.

### 3. IKE Parameters

Verify that both routers have matching:

- Encryption
- Hash
- Authentication
- Diffie-Hellman group
- Pre-shared key

### 4. IPSec Parameters

Verify that the transform sets match.

### 5. Crypto ACLs

The interesting-traffic ACLs should be mirror images.

### 6. Crypto Map

Confirm that the crypto map is applied to the correct WAN interface.

### 7. Routing

Verify that traffic can reach the remote LAN through the transit network.

### 8. Generate Interesting Traffic

The VPN may not establish until traffic matching the crypto ACL is generated.

For example:

```cisco
ping 10.1.2.x
```

Then check:

```cisco
show crypto isakmp sa
show crypto ipsec sa
```

---

# 📁 Repository Structure

```text
cisco-ipsec-site-to-site-vpn/
│
├── README.md
│
├── topology/
│   └── ipsec-site-to-site.png
│
├── configs/
│   ├── R1.txt
│   ├── R2.txt
│   └── ISP.txt
│
├── verification/
│   ├── show-crypto-isakmp-sa.txt
│   ├── show-crypto-ipsec-sa.txt
│   └── ping-tests.txt
│
├── documentation/
│   └── Cisco_IPSec_Site_to_Site_VPN_Lab.docx
│
└── notes/
    └── troubleshooting.md
```

---

# 📚 Documentation

The repository contains a detailed Word document covering the complete lab.

The article provides additional information about:

- VPN architecture
- IKE Phase 1
- IPSec Phase 2
- Crypto ACLs
- Crypto maps
- Routing
- IKE message exchange
- IPSec message exchange
- Packet flow
- Verification
- Troubleshooting
- Cisco configuration

The **README.md** is intended as the quick technical overview, while the Word document provides the deeper documentation.

---

# 💡 What This Lab Demonstrates

This project demonstrates practical understanding of:

- Cisco IOS configuration
- Site-to-site VPN architecture
- IKEv1
- IPSec
- ESP
- AES encryption
- SHA integrity
- Pre-shared-key authentication
- Diffie-Hellman key exchange
- Crypto ACLs
- Crypto maps
- VPN verification
- Network troubleshooting

---

# ⚠️ Lab vs Production

This project is designed for learning and portfolio purposes.

The configuration uses older IKEv1 parameters and a simple lab pre-shared key. A production deployment would require a security review and current organizational standards for:

- Cryptographic algorithms
- Key management
- Authentication
- Identity protection
- Logging
- Monitoring
- Device hardening
- Security policies

---

# 👨‍💻 Project Purpose

The purpose of this project is to demonstrate how a Cisco site-to-site IPSec VPN is configured, how IKE and IPSec work together, how interesting traffic triggers encryption, and how the resulting VPN tunnel can be verified and troubleshot.

This project forms part of my networking portfolio and demonstrates practical Cisco IOS VPN configuration and troubleshooting skills.
