# Lab 2 — Wireshark & Network Analysis


**Platform:** Wireshark (Free) · Local Machine or Azure VM
**Domain:** Network Analysis
**Status:** ✅ Complete

---

## Project Summary

This lab uses Wireshark to capture and analyze live network traffic — the same skill a network engineer, SOC analyst, or help-desk technician reaches for the moment a connectivity or security question can't be answered any other way. It walks through installing Wireshark, running a first capture, building fluency with display filters, and then four guided exercises: capturing a DNS lookup, watching a TCP three-way handshake, spotting cleartext credentials in an HTTP login, and following a full TCP stream to reconstruct a conversation between two hosts.

| Field | Value |
|---|---|
| Certification alignment | CompTIA Network+ · Security+ · CySA+ |
| Tools used | Wireshark (free, open source) · Npcap (Windows capture driver) |
| Time to complete | 2–4 hours across multiple sessions |
| Cost | $0 — Wireshark is permanently free |
| Career relevance | Network Engineer · SOC Analyst · Cloud Security Engineer · Incident Responder |

## 🎬 Watch Me Build This Lab!
https://www.loom.com/share/d581a1917e414740ab87fa632b2dc52d

---

## The Business Problem

Networks carry every piece of data an organization produces — emails, database queries, login credentials, file transfers, API calls. When something goes wrong — a service is unreachable, a user reports slow performance, a security alert fires — the network is almost always involved. The only way to know what is actually happening on a network is to look at the packets.

Wireshark is the tool organizations use to do that. It captures the raw data moving across a network interface and lets you inspect it at every layer — from the physical frame all the way up to the application payload.

### Where this shows up on the job

| Role | How this lab applies |
|---|---|
| Network Engineer | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| SOC Analyst | Identify malicious traffic patterns, extract indicators of compromise from packet captures |
| Cloud Security Engineer | The mental model from Wireshark transfers directly to reading Azure Network Watcher and VPC flow logs |
| Help Desk | Prove that a reported network issue is real and identify whether it is client-side or server-side |

---

## What This Lab Demonstrates

| Skill | Real-world application |
|---|---|
| Capture live network traffic | The foundational skill — you cannot analyze what you cannot see |
| Apply display filters | Production captures generate millions of packets. Filters let you find the 10 that matter in seconds |
| Read TCP handshakes | Recognizing a normal three-way handshake versus an incomplete one tells you immediately whether a connection succeeded or failed |
| Identify DNS queries and responses | DNS is involved in virtually every network action. Seeing it in packets gives you a mental model that transfers to cloud DNS troubleshooting |
| Spot cleartext credentials in HTTP | A hands-on demonstration of why HTTPS matters — you will see actual credentials in plaintext |
| Follow TCP streams | Reassemble the full conversation between two hosts from individual packets — essential for incident investigation |

---

## Architecture — How Wireshark Captures Traffic

```
Internet
   │
   ▼
Router / Default Gateway
   │
   ▼
Local Network Interface (Ethernet or Wi-Fi)
   │  ← NIC set to promiscuous mode by Wireshark
   ▼
Wireshark (libpcap/Npcap capture driver)
   │
   ├── Live packet list  (every frame, in real time)
   ├── Display filters    (dns / http / tcp.flags.syn==1 / ip.addr==x.x.x.x ...)
   ├── Packet detail pane (drill into headers: Ethernet → IP → TCP/UDP → payload)
   └── Follow TCP Stream  (reassembled conversation, request in red / response in blue)
```

**Key mechanism:** normally a NIC only accepts frames addressed to it. Wireshark switches the interface into **promiscuous mode**, so the capture driver (Npcap on Windows) hands every frame the NIC sees to Wireshark — not just this machine's own traffic. On a modern switched network you'll mostly see your own traffic plus broadcasts; port mirroring or a hub is required to see other hosts' traffic.

---

## Walkthrough & Screenshots

Screenshots below are placeholders — drop the corresponding image into `screenshots/` using the filenames shown and they will render here automatically.

### 1. Wireshark and Npcap installed

<img width="1458" height="867" alt="Image" src="https://github.com/user-attachments/assets/5691ac0c-ac70-4e3b-ac25-912f44b157c4" />

### 2. First capture stopped — full packet list

![Packet list after first capture](<img width="729" height="793" alt="Image" src="https://github.com/user-attachments/assets/911baa42-b3f6-46e3-926e-24b483dbc24e" />

### 3. Exercise A — DNS query and response for google.com

![DNS query and response]<img width="1165" height="322" alt="Image" src="https://github.com/user-attachments/assets/e44dfb23-5089-44ae-9024-165fe2cfb6f6" />

### 4. Exercise B — TCP three-way handshake (SYN → SYN-ACK → ACK)

![TCP three-way handshake]<img width="729" height="793" alt="Image" src="https://github.com/user-attachments/assets/23796a5a-c672-48fe-89c1-2eb3fa41c6ed" />

### 5. Exercise D — Follow TCP Stream (reassembled conversation)

![Follow TCP stream] <img width="896" height="825" alt="Image" src="https://github.com/user-attachments/assets/2b8bafb6-930a-44ed-9aa4-c61743c1be39" />

### 6. Captures saved as .pcapng

![Saved pcapng captures]<img width="898" height="758" alt="Screenshot 2026-07-19 at 9 57 57 AM" src="https://github.com/user-attachments/assets/fc5af469-3829-40f8-83c0-d5393e1f6e53" />


---

## Verification

| Check | How to verify | Expected result |
|---|---|---|
| DNS capture | Apply `dns` filter and identify a query packet and its response packet | Matching transaction IDs; response contains the A record IP |
| TCP handshake | Find three sequential packets with SYN, SYN-ACK, ACK flags | Can explain what each packet means without notes |
| Display filters | Filter by IP address, port, and protocol from memory | No lookup needed — `ip.addr==`, `tcp.port==`, `dns`, `http` recalled directly |
| Stream reconstruction | Follow a TCP stream and read the full HTTP request/response | Red = request, blue = response, readable as a conversation |
| File management | Save a capture, close Wireshark, reopen it, and load the file | All packets present after reopening |

All checks passed — see `scripts/05-Verify-Captures.ps1` for the runnable version of the file-management check.

---

## Repo Contents

```
lab-2-wireshark/
├── README.md                              ← this file
├── SOP-runbook.md                         ← step-by-step operational procedure
├── screenshots/                           ← drop lab screenshots here (see filenames above)
├── captures/                              ← .pcapng files saved during the lab (dns-lookup, tcp-handshake, tcp-stream)
└── scripts/
    ├── 01-Install-Wireshark.ps1
    ├── 02-Capture-DNS-Lookup.ps1
    ├── 03-Capture-TCP-Handshake.ps1
    ├── 04-Capture-HTTP-Cleartext-Demo.ps1
    └── 05-Verify-Captures.ps1
```

## Lessons Learned / Notes

- Display filters (`dns`, `tcp.flags.syn==1`) are applied **after** capture and never discard data — capture filters are applied before capture and do discard data. For this lab, always capture everything and filter the view afterward; you can't go back and re-capture what a capture filter already dropped.
- `nslookup` and any browser navigation must run in a **separate terminal/browser window** from Wireshark — Wireshark is capture/analysis only, it has no terminal of its own.
- Run `nslookup example.com` *before* filtering the TCP handshake capture — you need the resolved IP to build the `tcp and ip.addr == <ip>` filter in Exercise B.
- Exercise C (cleartext credentials) is an educational demonstration only — only capture traffic on networks and systems you own or have explicit permission to analyze.
- `tshark` (bundled with Wireshark) is useful for headless/remote captures where there's no GUI — e.g., on an Azure VM accessed only via SSH/RDP with no desktop session.
