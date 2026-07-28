# uShield 🛡️
### A Unified Hardware-Based Security System for Small-to-Medium Enterprise Networks

> **Zagazig University — Faculty of Engineering**
> Electronics & Communications Engineering — Class of 2026
> Supervised by: Dr. Nirmeen Monir

---

## What is uShield?

uShield is a graduation project that implements a complete, layered network security system using affordable, accessible hardware and open-source software — designed to bring enterprise-grade protection to small and medium-sized enterprises (SMEs) that cannot afford commercial security appliances.

The name stands for **Unified Shield** — every security layer (switching, firewall, IDS, IoT, dashboard) works together as one coordinated system rather than isolated components.

The project proves that strong network security is not a function of hardware cost. It is a function of correct architecture, correct configuration, and correct integration.

---

## The Problem

Small and medium enterprises handle sensitive data but lack access to affordable, integrated network security solutions. Enterprise-grade UTM appliances and IDS platforms are too expensive and too complex. Consumer routers are too simplistic. The result: most SMEs operate on flat, unmonitored networks with no VLAN segmentation, no intrusion detection, no IoT isolation, and no meaningful firewall policy.

A single compromised device — a camera, a laptop, an unpatched sensor — can move laterally across the entire network undetected.

---

## The Solution

uShield builds a unified security system from accessible components:

| Component | Hardware | Role |
|---|---|---|
| Access Layer Switch | Cisco Catalyst WS-C2960-24PC-L | VLAN segmentation, LAN security hardening, traffic mirroring |
| Firewall & Gateway | Raspberry Pi 5 Model B (8 GB) + OpenWRT 24.10.3 | Stateful firewall, inter-VLAN routing, DHCP, DNS, NAT |
| Intrusion Detection | IDS Laptop running Suricata | Passive traffic analysis, EVE JSON alert generation |
| IoT Layer | ESP32 + Uniview IP Camera + Sensors | Physical security simulation — camera, door lock, environment |
| Web Dashboard | Python CGI + HTML5 + Chart.js on uhttpd | Simplified admin interface — IDS alerts, IP blocking, monitoring |
| Attack Simulation | Kali Linux VM | Live penetration testing to validate all defenses |

---

## System Architecture

```
           [ Internet ]
                |
          [ ISP Router ]
                |
   [ Raspberry Pi 5 — OpenWRT ]
     Firewall | DHCP | DNS | NAT
        |               |
  [ Wi-Fi AP ]    [ Cisco Catalyst 2960 ]
                  802.1Q Trunk (Gi0/1)
                        |
        ┌───────┬───────┬───────┬───────┐
     [VLAN 10] [VLAN 20] [VLAN 30] [VLAN 100] [VLAN 136]
       IT        HR      Servers    IoT       Clients
                                    |
                              [IDS — Fa0/24]
                              SPAN Mirror Port
                                    |
                            [ Web Dashboard ]
```

### Network Segmentation

| VLAN | Name | Ports | Subnet |
|---|---|---|---|
| 10 | IT Department | Fa0/1–4 | 192.168.10.0/24 |
| 20 | HR Department | Fa0/5–8 | 192.168.20.0/24 |
| 30 | Servers | Fa0/9–12 | 192.168.30.0/24 |
| 100 | IoT Zone | Fa0/13–16 | 192.168.100.0/24 |
| 136 | Clients | Fa0/17–19 | 192.168.136.0/24 |
| 666 | Black Hole (unused) | Fa0/20–23 | No IP — shutdown |
| 25 | IDS Mirror | Fa0/24 | SPAN destination only |
| 999 | Native Trunk | Gi0/1 | Anti-VLAN-hopping |

---

## Security Features

### Switch Layer (Cisco Catalyst 2960)
- ✅ Full VLAN segmentation across 5 isolated zones
- ✅ DHCP Snooping with rate limiting (4 packets/sec per port)
- ✅ Dynamic ARP Inspection (DAI) — prevents ARP poisoning / Man-in-the-Middle
- ✅ PVST+ with PortFast & BPDU Guard — STP hardening, rogue switch prevention
- ✅ Unused ports shutdown + assigned to Black Hole VLAN 666
- ✅ SSH-only VTY management, local user authentication
- ✅ Console password + MOTD banner + enable secret + service password-encryption
- ✅ SPAN session: mirrors all trunk traffic (Gi0/1) → IDS port (Fa0/24)
- ✅ Native VLAN 999 on trunk — hardens against double-tagging VLAN hopping

### Firewall & Gateway (Raspberry Pi 5 + OpenWRT 24.10.3)
- ✅ Stateful firewall with default-DENY policy
- ✅ HTTP blocked — HTTPS only enforced
- ✅ DNS enforcement — OpenWRT is the sole resolver, no bypass possible
- ✅ TCP connection rate limiting
- ✅ Inter-VLAN default deny — all cross-VLAN flows explicitly permitted or blocked
- ✅ Management interface restricted to authorized VLAN only
- ✅ Martian / bogon packet detection and logging
- ✅ Automated IP blocking on brute-force detection
- ✅ AdBlock (LuCI package) — DNS-layer domain blocking
- ✅ DHCP server for all 5 VLANs + NAT

### Intrusion Detection System (Suricata)
- ✅ Passive monitoring via SPAN mirror port — zero network impact
- ✅ Signature-based detection with EVE JSON structured output
- ✅ Flask API exposes alerts to dashboard in real time
- ✅ Zero false positives confirmed during normal operation baseline

### IoT Layer
- ✅ IP camera — Full HD, PoE, Smart IR, ONVIF (Uniview IPC3612LB-AF28-ECO)
- ✅ Smart door lock (ESP32) — PIN keypad + RFID card, servo motor, LCD, buzzer
- ✅ Environmental sensors — temperature & pressure, live dashboard reporting
- ✅ All IoT devices isolated in VLAN 100
- ✅ Replay attack protection confirmed
- ✅ Default credentials changed and audited on all devices

### Web Dashboard
- ✅ Real-time IDS alert panel
- ✅ Attack-type doughnut chart (Chart.js)
- ✅ One-click IP blocking → UCI → nftables applied instantly
- ✅ Accessible only from authorized management VLAN
- ✅ HTTPS enforced — no plaintext HTTP access

---

## Testing Results — 21/21 PASS

21 test cases across 17 attack scenarios in 3 phases using Kali Linux.

| Test Case | Description | Result |
|---|---|---|
| TC-01 | VLAN Isolation | ✅ PASS |
| TC-02 | Port Scan — All Ports Filtered | ✅ PASS |
| TC-03 | Controlled DoS — No Service Disruption | ✅ PASS |
| TC-04 | Brute Force → Auto IP Block | ✅ PASS |
| TC-05 | Cross-VLAN Command Injection Blocked | ✅ PASS |
| TC-06 | Outbound Exfiltration Blocked | ✅ PASS |
| TC-07 | Unused Port Exposure — All Filtered | ✅ PASS |
| TC-08 | Spoofed Data Injection Blocked | ✅ PASS |
| TC-09 | Power Failure — All Rules Persisted | ✅ PASS |
| TC-10 | Management Interface — Access Denied | ✅ PASS |
| TC-11 | MAC Spoofing Blocked | ✅ PASS |
| TC-12 | Martian Packet Dropped & Logged | ✅ PASS |
| TC-13 | HTTP Blocked / HTTPS Enforced | ✅ PASS |
| TC-14 | Rogue DHCP Server Blocked | ✅ PASS |
| TC-15 | Default Credentials Rejected | ✅ PASS |
| TC-16 | IDS Signature Alert Triggered | ✅ PASS |
| TC-17 | Replay Attack Rejected | ✅ PASS |
| TC-18 | TLS Audit — Strong Ciphers Only | ✅ PASS |
| TC-19 | VLAN Hopping (Double-Tagging) Blocked | ✅ PASS |
| TC-20 | IDS — Zero False Positives Baseline | ✅ PASS |
| TC-21 | Wireless Clients Isolated | ✅ PASS |

---

## Hardware Specifications

### Cisco Catalyst WS-C2960-24PC-L
- 24x FastEthernet PoE + 2x GigabitEthernet
- IOS: 12.2(55)SE12

### Raspberry Pi 5 Model B
- Quad-core Arm Cortex-A76 @ 2.4 GHz — 8 GB RAM — 64 GB microSD
- OS: OpenWRT 24.10.3

### Uniview IPC3612LB-AF28-ECO
- 2 MP Full HD 1080p / 30fps / Smart IR 30m / PoE / RTSP / ONVIF

### ESP32 DevKit (Door Lock Controller)
- Dual-core Xtensa LX6 @ 240 MHz — 34 GPIO — Wi-Fi + BT
- Peripherals: W5500 Ethernet, I2C LCD 16x2, 4x4 Keypad,
  RC522 RFID, HC-SR04 Ultrasonic, MG996R Servo, Active Buzzer

---

## Software Stack

| Software | Version | Purpose |
|---|---|---|
| OpenWRT | 24.10.3 | Firewall OS on Raspberry Pi |
| Suricata | bundled | Network IDS |
| Flask | Python | IDS alert REST API |
| Python CGI + uhttpd | Python 3 | Dashboard backend + web server |
| UCI / firewall4 / nftables | built-in | Firewall rule engine |
| AdBlock | LuCI package | DNS-layer domain blocking |
| HTML5 / CSS3 / Chart.js | Web standards | Dashboard frontend |
| Arduino IDE | Latest | ESP32 firmware development |
| Kali Linux | Latest | Attack simulation & penetration testing |

---

## Repository Structure

```
uShield/
├── README.md
├── docs/
│   ├── uShield_Book.pdf
│   ├── uShield_Presentation.pdf
│   └── uShield_Master_Structure.pdf
├── switch/
│   ├── startup_config.txt
│   └── README.md
├── firewall/
│   ├── firewall_rules.txt
│   ├── network_config.txt
│   └── README.md
├── ids/
│   ├── suricata_config.yaml
│   ├── sample_eve_alert.json
│   └── README.md
├── dashboard/
│   ├── index.html
│   ├── style.css
│   ├── app.js
│   ├── backend.py
│   └── README.md
├── iot/
│   ├── door_lock/
│   │   ├── esp32_doorlock.ino
│   │   └── wiring_diagram.png
│   ├── sensors/
│   │   └── sensors.ino
│   └── README.md
├── diagrams/
│   ├── network_topology.png
│   ├── traffic_flow.png
│   └── vlan_design.png
└── testing/
    ├── test_cases_table.pdf
    └── results_summary.md
```


## Future Work

- 🤖 **AI-Powered Autonomous Security** — AI classifies IDS alerts, auto-executes responses, eliminates need for a security engineer
- 🔒 **Inline IDS → IPS Mode** — Suricata moves inline to drop malicious traffic in real time
- 📊 **Full SIEM Dashboard** — aggregate all component logs, anomaly detection, scheduled reports
- 🌐 **VPN Tunneling** — WireGuard on OpenWRT for secure remote access
- 📡 **Wireless Intrusion Detection** — RF monitoring for rogue APs and deauth attacks
- 🖥️ **Dedicated Firewall Appliance** — replace Raspberry Pi with pfSense/OPNsense for production scale
- 📈 **Horizontal Scaling** — additional VLANs, stacked switches, expanded IoT fleet

---

## Acknowledgements


**Faculty of Engineering — Zagazig University**
Electronics & Communications Engineering — Class of 2026

---

> *"Security is not a product. It is a discipline."*
> *uShield — built to prove that enterprise-grade network security is accessible to everyone.*
