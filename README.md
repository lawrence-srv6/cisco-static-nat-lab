# Cisco Cisco IOS Static NAT Configuration & Troubleshooting Lab

An entry-level networking lab focused on implementing 1-to-1 Static Network Address Translation (NAT) and troubleshooting Layer 3 misconfigurations using Cisco IOS.

## 🗺️ Topology Overview
* **Internal Private Subnet:** 172.16.0.0/24 (PCs 1-3)
* **WAN / ISP Segment:** 203.0.113.0/30
* **Public NAT IP Block:** 100.0.0.0/24 (Mapped 1-to-1 for local hosts)
* **Target External Server:** 8.8.8.8 (Internet Simulation)

## 🛠️ Tech Stack & Commands Used
* **Simulation Software:** Cisco Packet Tracer
* **Core Protocols:** IPv4 Routing, Static NAT, Static Default Routing, ICMP, ARP
* **Key Verification Commands:** `show ip nat translations`, `show ip nat statistics`, `show ip interface brief`

## 🧠 What I Learned & Troubleshooting Journey

### 1. The Problem
After implementing the basic IP infrastructure and default routes, initiating a ping from **PC1 (172.16.0.1)** to the internet server (**8.8.8.8**) failed with a timeout error.

### 2. Isolation Workflow
* **Layer 2 Verification:** Confirmed local physical states and MAC addresses using `show ip arp`. 
* **Layer 3 Verification:** Used `tracert` and **Packet Tracer Simulation Mode** to track packet flow. The packet reached the external gateway but failed on return processing.
* **NAT Auditing:** Ran `show ip nat translations`. The static mappings were sitting in the table correctly, but traffic wasn't generating active hits.

### 3. The "Aha!" Moment & Resolution
By executing `show running-config`, I discovered a muscle-memory mistake: I had accidentally swapped the NAT interface directions. The LAN interface was designated as `ip nat outside` and the WAN interface as `ip nat inside`.

Once inverted using the correct parameters, end-to-end connectivity succeeded instantly:
```text
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip nat inside
R1(config-if)# interface GigabitEthernet0/0
R1(config-if)# ip nat outside
```

## 📋 Final Working R1 NAT Configuration
```text
interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252
 ip nat outside
!
interface GigabitEthernet0/1
 ip address 172.16.0.254 255.255.255.0
 ip nat inside
!
ip nat inside source static 172.16.0.1 100.0.0.1
ip nat inside source static 172.16.0.2 100.0.0.2
ip nat inside source static 172.16.0.3 100.0.0.3
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```
