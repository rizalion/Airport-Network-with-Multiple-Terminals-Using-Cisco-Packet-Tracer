# Airport Network with Multiple Terminals

A simulated enterprise-scale airport network designed and implemented using **Cisco Packet Tracer**, featuring multiple terminals, VLAN segmentation, inter-VLAN routing, centralized network services, dynamic routing, and ACL-based security.

This project was developed as a semester project for **Data Communication & Computer Networks (DCCN)** at **HITEC University, Taxila**.

---

## Project Overview

The project models a realistic airport network consisting of:

* **Terminal A**
* **Terminal B**
* **Terminal C**
* **Central Core / Data Center**

Each terminal contains separate network segments for:

* Operations
* Passenger Services
* Security

A centralized server farm provides essential network services including:

* DHCP
* DNS
* Web/HTTP
* Email

The network uses **Router-on-a-Stick**, **RIPv2**, **VLANs**, **ACLs**, **STP**, subnetting, and both copper and fiber connections.

---

## Objectives

The main objectives of the project were to:

* Design a hierarchical and scalable multi-terminal network.
* Implement VLAN segmentation for departmental isolation.
* Configure centralized DHCP services.
* Deploy DNS, Web, and Email servers.
* Implement inter-VLAN routing using Router-on-a-Stick.
* Configure RIPv2 dynamic routing.
* Implement ACLs to restrict unauthorized traffic.
* Design an IP addressing scheme capable of supporting 2,000+ devices.
* Test end-to-end connectivity and network services.
* Troubleshoot common networking issues in a simulated enterprise environment.

---

## Network Architecture

The network follows a hierarchical architecture consisting of three logical layers.

### Core Layer

The central network contains:

* `CORE-RT-R` — Cisco 2911 Core Router
* `CORE-SW` — Cisco 2960-24TT Core Switch
* DHCP Server
* DNS Server
* Web Server
* Email Server
* Management PC

### Terminal Layer

Each terminal contains:

* One Cisco 2911 router
* One Cisco 2960-24TT switch
* Operations PC
* Passenger Services PC
* Security PC

The terminal routers provide inter-VLAN routing and exchange routes with the core using RIPv2.

---

## VLAN Design

Three VLANs are implemented at each terminal.

| VLAN | Name       | Department         | Purpose                                         |
| ---- | ---------- | ------------------ | ----------------------------------------------- |
| 10   | OPERATIONS | Operations         | Flight scheduling, ground operations, logistics |
| 20   | PASSENGERS | Passenger Services | Ticketing, boarding, passenger information      |
| 30   | SECURITY   | Security           | Surveillance, access control, security alerts   |

VLAN segmentation reduces broadcast domains and provides logical separation between departments.

---

## IP Addressing

### Terminal A

| VLAN    | Network         | Gateway      |
| ------- | --------------- | ------------ |
| VLAN 10 | `10.10.10.0/24` | `10.10.10.1` |
| VLAN 20 | `10.20.20.0/24` | `10.20.20.1` |
| VLAN 30 | `10.30.30.0/24` | `10.30.30.1` |

### Terminal B

| VLAN    | Network         | Gateway      |
| ------- | --------------- | ------------ |
| VLAN 10 | `10.10.11.0/24` | `10.10.11.1` |
| VLAN 20 | `10.20.21.0/24` | `10.20.21.1` |
| VLAN 30 | `10.30.31.0/24` | `10.30.31.1` |

### Terminal C

| VLAN    | Network         | Gateway      |
| ------- | --------------- | ------------ |
| VLAN 10 | `10.10.12.0/24` | `10.10.12.1` |
| VLAN 20 | `10.20.22.0/24` | `10.20.22.1` |
| VLAN 30 | `10.30.32.0/24` | `10.30.32.1` |

### Server Network

| Device       | IP Address       | Service                |
| ------------ | ---------------- | ---------------------- |
| Core Router  | `192.168.100.1`  | Server Network Gateway |
| DHCP Server  | `192.168.100.10` | DHCP                   |
| DNS Server   | `192.168.100.20` | DNS                    |
| Web Server   | `192.168.100.30` | HTTP                   |
| Email Server | `192.168.100.40` | SMTP/POP3              |

### Router Interconnections

| Link              | Network         | Core IP      | Terminal IP  |
| ----------------- | --------------- | ------------ | ------------ |
| Core ↔ Terminal A | `172.16.1.0/30` | `172.16.1.1` | `172.16.1.2` |
| Core ↔ Terminal B | `172.16.2.0/30` | `172.16.2.1` | `172.16.2.2` |
| Core ↔ Terminal C | `172.16.3.0/30` | `172.16.3.1` | `172.16.3.2` |

The design provides a theoretical capacity of approximately **2,546 usable IP addresses** across the configured networks.

---

## Network Services

### DHCP

A centralized DHCP server automatically provides:

* IP addresses
* Subnet masks
* Default gateways
* DNS server information

DHCP relay is implemented using:

```text
ip helper-address 192.168.100.10
```

on the terminal router sub-interfaces.

### DNS

The DNS server provides internal name resolution for:

```text
www.airport.com
mail.airport.com
```

Example:

```text
www.airport.com → 192.168.100.30
mail.airport.com → 192.168.100.40
```

### Web Server

The internal airport website is hosted on:

```text
192.168.100.30
```

and accessed through:

```text
http://www.airport.com
```

### Email Server

The email server uses the:

```text
airport.com
```

domain and provides internal communication between airport staff.

---

## Routing

### Router-on-a-Stick

Each terminal router uses 802.1Q sub-interfaces to route traffic between VLANs.

Example:

```text
interface g0/1.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
 ip helper-address 192.168.100.10
```

Separate sub-interfaces are configured for VLANs 10, 20, and 30.

### RIPv2

RIPv2 is configured across the core and terminal routers for dynamic route exchange.

Example:

```text
router rip
 version 2
 network 10.0.0.0
 network 172.16.0.0
 no auto-summary
```

The use of `no auto-summary` allows the configured subnet structure to be advertised without classful summarization.

---

## Security

An extended ACL is used to restrict traffic from the Passenger VLAN to the Security VLAN.

Example:

```text
access-list 100 deny ip 10.20.20.0 0.0.0.255 10.30.30.0 0.0.0.255
access-list 100 permit ip any any
```

The ACL is applied inbound on the Passenger VLAN interface:

```text
interface g0/1.20
 ip access-group 100 in
```

This demonstrates how network access policies can be enforced at the routing layer.

---

## Spanning Tree Protocol

STP is used on the switches to maintain a loop-free Layer 2 topology and prevent switching loops and broadcast storms.

Verification can be performed with:

```text
show spanning-tree
```

---

## Testing & Validation

The network was tested using multiple connectivity and service-validation techniques.

### Connectivity Tests

* Terminal A → Terminal B
* Terminal A → Terminal C
* Terminal B → Terminal C
* Terminal PCs → Web Server
* Terminal PCs → DNS Server
* Terminal PCs → DHCP Server

### Service Tests

* DHCP address acquisition
* DNS name resolution
* Web server access
* Email service configuration
* Inter-VLAN communication

### Security Tests

Passenger-to-Security traffic was tested to verify ACL enforcement.

Expected behavior:

```text
Passenger VLAN → Security VLAN
             ↓
           DENIED
```

### Routing Tests

The following commands were used for verification:

```text
show ip route
show ip interface brief
show access-lists
show spanning-tree
```

Traceroute was also used to verify the expected path:

```text
PC
 ↓
Switch
 ↓
Terminal Router
 ↓
Core Router
 ↓
Server Network
```

---

## Troubleshooting

Several networking issues were encountered and resolved during implementation.

| Problem                          | Cause                                     | Solution                                       |
| -------------------------------- | ----------------------------------------- | ---------------------------------------------- |
| DHCP failure                     | Missing DHCP relay/configuration          | Configured DHCP pools and `ip helper-address`  |
| DNS resolution failure           | Incorrect DNS configuration/A-records     | Corrected DNS settings and records             |
| Terminal C link failure          | Incorrect module/cable configuration      | Installed GLC-LH-SMD and used fiber            |
| Inter-VLAN communication failure | Missing trunk/sub-interface configuration | Configured 802.1Q trunks and Router-on-a-Stick |

These troubleshooting steps provided practical experience with diagnosing Layer 2, Layer 3, and service-level networking problems.

---

## Hardware & Technologies

### Network Devices

* Cisco 2911 Routers
* Cisco 2960-24TT Switches
* Server-PT
* PC-PT

### Technologies

* IPv4
* Subnetting
* VLANs
* 802.1Q
* Router-on-a-Stick
* DHCP
* DHCP Relay
* DNS
* HTTP
* SMTP/POP3
* RIPv2
* Static Routing
* Extended ACLs
* STP
* TCP/IP

### Cables

* Copper Straight-Through
* Copper Cross-Over
* Fiber Optic

### Software

* Cisco Packet Tracer

---

## Project Structure

A recommended repository structure is:

```text
airport-network-cisco-packet-tracer/
│
├── README.md
│
├── packet-tracer/
│   └── airport_network.pkt
│
├── documentation/
│   └── project-design-report.pdf
│
├── screenshots/
│   ├── topology.png
│   ├── vlan-verification.png
│   ├── routing-table.png
│   ├── acl-verification.png
│   ├── dhcp-configuration.png
│   ├── dns-configuration.png
│   ├── email-configuration.png
│   ├── ping-tests.png
│   ├── dns-test.png
│   └── web-server-test.png
│
└── results/
    └── testing-results.md
```

> The `.pkt` file and screenshots should contain the actual Cisco Packet Tracer project and evidence from the implementation.

---

## Learning Outcomes

This project provided practical experience in:

* Enterprise network design
* IPv4 subnetting
* VLAN segmentation
* Inter-VLAN routing
* Dynamic routing
* DHCP relay
* DNS configuration
* Server deployment
* ACL-based traffic control
* STP
* Network troubleshooting
* End-to-end connectivity testing
* Cisco IOS configuration

---

## Future Improvements

Possible extensions include:

* Replacing RIPv2 with OSPF
* Adding redundant links
* Implementing HSRP for gateway redundancy
* Adding wireless access points for passenger Wi-Fi
* Introducing SNMP and Syslog monitoring
* Adding redundant/high-availability servers
* Expanding the network with additional terminals
* Implementing more granular ACL policies

---

## Project Information

**Course:** Data Communication & Computer Networks (DCCN)
**Institution:** HITEC University, Taxila
**Semester:** Spring 2026 — 6th Semester
**Section:** 6C
**Instructor:** Syeda Hina Gillani
**Project Type:** Semester Project
**Simulation Tool:** Cisco Packet Tracer

### Author

**Muhammad Huzaifa — 23-CS-007**

---

## Disclaimer

This project is an educational network simulation created in Cisco Packet Tracer. It demonstrates enterprise networking concepts in a controlled virtual environment and is not intended to represent a production airport infrastructure.

---
