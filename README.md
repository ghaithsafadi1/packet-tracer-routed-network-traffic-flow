# Packet Tracer – Observe Traffic Flow in a Routed Network

## Lab Overview

This lab demonstrates how routing between separate department networks improves network efficiency in an enterprise environment. Using Cisco Packet Tracer's simulation mode, we observe and compare traffic behavior in an **unrouted LAN** versus a **routed multi-network topology**.

The scenario is based on a growing company (XYZ LLC) whose single flat network is causing performance degradation as the number of devices increases.

---

## Objectives

- **Part 1:** Observe traffic flow in an unrouted LAN
- **Part 2:** Reconfigure the network to route between LANs
- **Part 3:** Observe traffic flow in the routed network

---

## Network Topology

The topology consists of:

- **Edge Router** – connects the internal LAN to the internet (ISP cloud)
- **Three department switches** – Accounting, Finance, and Sales
- **Multiple hosts** per department (e.g., Sales 1, Sales 2)

> **Note:** The Packet Tracer topology is simplified for demonstration purposes. In a real-world scenario, the behavior shown here scales to 150+ devices.

---

## Part 1 – Unrouted LAN Behavior

### What Happens

In the original design, all departments share a **single IPv4 network**. The Edge Router only routes traffic between the internal LAN and the internet — it does **not** route between departments.

### Steps Performed

1. Cleared the ARP cache on **Sales 1** using `arp -d`
2. Switched to **Simulation Mode** in Packet Tracer
3. Pinged Sales 1 from Sales 2 and stepped through the PDUs

### Key Observations

| Question | Answer |
|----------|--------|
| Source/Destination MAC and IP of first PDU? | Source = Sales 2 MAC/IP; Destination = **Broadcast** MAC (`FF:FF:FF:FF:FF:FF`) / Sales 1 IP |
| Why broadcast MAC? | Sales 2 doesn't know Sales 1's MAC — it sends an **ARP request** to the broadcast address to resolve it |
| Which devices processed the ARP request? | **All hosts and switches** in the network, across all departments |
| Impact on network efficiency? | ARP broadcasts flood the entire network — every device must process them, causing unnecessary overhead and congestion |
| Type of the second PDU (different color)? | **ICMP Echo Request** (the actual ping) — sent after ARP resolved the MAC address |

### Problem with a Flat Network

When all devices share one network, every ARP request is broadcast to every device in the LAN. In a 150-device network, this generates significant unnecessary traffic across all departments — a problem that worsens as the network grows.

---

## Part 2 – Reconfiguring to a Routed Network

### Cable Changes

The switches were originally daisy-chained (Accounting → Finance → Sales). They were re-cabled so that **each switch connects directly to a dedicated router interface**:

| Switch | Router Interface |
|--------|-----------------|
| Accounting | GigabitEthernet 0/0 |
| Finance | GigabitEthernet 1/0 |
| Sales | GigabitEthernet 2/0 |

### IP Address Reassignment

Hosts on the Finance and Sales networks requested new addresses via DHCP:

```
ipconfig /renew
```

The Edge Router (acting as DHCP server) assigned hosts to new subnets:

| Department | IPv4 Network |
|------------|-------------|
| Accounting | `192.168.1.0/24` *(unchanged)* |
| Finance | `192.168.2.0/24` *(example)* |
| Sales | `192.168.3.0/24` *(example)* |

> Actual subnet assignments may vary depending on the router's DHCP pool configuration in your `.pkt` file.

---

## Part 3 – Routed Network Behavior

### Steps Performed

1. Cleared ARP cache on Sales 2
2. Switched to Simulation Mode
3. Pinged Sales 1 from Sales 2 and observed ARP traffic

### Key Observations

| Question | Answer |
|----------|--------|
| Which devices receive ARP broadcasts now? | Only devices **within the Sales subnet** — the router does not forward broadcasts across network boundaries |
| Benefit of multiple IPv4 networks? | Broadcast traffic is **contained within each subnet**, reducing unnecessary load on devices in other departments |

### Why This Matters

Routers create **broadcast domain boundaries**. Each department network is isolated, so:

- An ARP request from Sales only reaches Sales devices
- Finance and Accounting devices are completely unaffected
- Network efficiency improves significantly as the number of devices grows

---

## Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| **ARP (Address Resolution Protocol)** | Resolves IP addresses to MAC addresses using Layer 2 broadcasts |
| **Broadcast Domain** | The set of devices that receive a broadcast frame — limited by routers |
| **DHCP** | Dynamically assigns IP addresses; used here to move hosts to new subnets |
| **Routing** | Forwarding of packets between different IP networks via a router |
| **Network Segmentation** | Dividing one large flat network into smaller subnets to improve performance |

---

## Tools Used

- **Cisco Packet Tracer** – Network simulation
- **Simulation Mode** – Step-by-step PDU inspection
- **CLI Commands:**
  - `arp -a` – View ARP cache
  - `arp -d` – Clear ARP cache
  - `ping <IP>` – Test connectivity
  - `ipconfig /renew` – Request new DHCP address

---

## Key Takeaway

> Implementing routing between department subnets **contains broadcast traffic** within each network segment. This reduces unnecessary processing on all devices, decreases network congestion, and improves overall efficiency — especially critical as organizations scale.

---

## Lab File

The Packet Tracer `.pkt` file for this lab can be found in the root of this repository.

---

*Part of my CCNA study portfolio — built using Cisco Packet Tracer and NetAcad curriculum.*
