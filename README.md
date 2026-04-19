# Antonio Alfano

**Site Reliability Engineer · Infrastructure Architect · Hardware Engineer**

Edmonton, Alberta · Dual 🇺🇸 / 🇨🇦 Citizen · No sponsorship required

[![Portfolio](https://img.shields.io/badge/Portfolio-alfanotechport.org-0A66C2?style=flat-square)](https://alfanotechport.org)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Antonio_Alfano-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/antonio-alfano-273162358)
[![Status](https://img.shields.io/badge/Infrastructure-Live_Production-22C55E?style=flat-square)]()
[![Uptime](https://img.shields.io/badge/SLA_Target-99.9%25-22C55E?style=flat-square)]()

---

## About

I build and operate infrastructure that does not fail gracefully because it is engineered not to fail at all. My work spans the full stack from BGA rework on ASIC hashboards to multi-tenant production hosting platforms, with a strong focus on automation, observability, and zero trust architecture.

I currently own end to end Site Reliability Engineering for five distributed compute facilities across Alberta, totaling **300+ PH/s of managed capacity**, and I run a freelance hosting business (Alfano Tech) off a Proxmox homelab that I designed, built, and automated from the ground up. Before engineering I played defensive line in the CFL for the Edmonton Elks. The discipline transferred cleanly.

I am not a beginner. I do not use drag and drop site builders, I do not resell AWS, and I do not ship infrastructure I cannot diagnose at Layer 1.

---

## What I Do Well

### Production SRE at scale
Five sites. 300+ PH/s. 24/7 availability. Dynamic load shedding and three phase power balancing across climate swings from 30 below to 95 above. I own the pager.

### Infrastructure automation
I have built the full provisioning pipeline for a multi tenant hosting platform, zero to live in under five minutes, nine automated steps covering LXC creation, VLAN assignment, nginx vhost, reverse proxy wiring, Cloudflare Tunnel ingress injection via API, DNS, Prometheus scrape target registration, and Uptime Kuma monitor creation. No manual dashboard clicks.

### Observability
Full LGTM stack in production (Grafana, Loki, Tempo, Mimir, Prometheus, Node Exporter) with alert rules tuned for real failure modes, not textbook ones. Four live Grafana rules routed to SMTP, every one correctly handling the "no data" case that most engineers miss.

### Hardware down to the silicon
Fluke multimeter, oscilloscope, hot air station, microscope. I trace 1.8V, 0.8V, and 12V voltage rails on hashboard control boards, identify shorted MOSFETs and dead LDO regulators, and perform BGA rework to recover dead capacity. I crimp my own Cat 5e through Cat 8, and I can diagnose packet loss at the physical layer with an OTDR.

### AI engineering and local inference
I run CPU inference workloads via llama.cpp on Node 1 (Ryzen 9 9950X3D, 64 GB DDR5 6000), pinning NUMA regions and thread counts to squeeze useful tokens per second out of hardware that shouldn't be able to do it. I also work extensively with MCP (Model Context Protocol) servers to extend LLM capability into real infrastructure operations, treating AI as a teammate with tools, not as a chatbot.

---

## Flagship Project: Alfano Tech (Project Titan)

A fully automated, self-hosted, multi-tenant web hosting platform running on a Proxmox homelab in Edmonton. Every decision was made to maximize availability per dollar spent and keep operational overhead near zero.

### Architecture overview

```
Internet
   │
   ▼
Cloudflare (Tunnel, Workers, DNS, SSL, Pages failover)
   │
   ▼  ingress only, zero open inbound ports
OPNsense firewall (VM 110, 4c/16GB)
   │
   ▼  VLAN tagged
Proxmox VE 9.1.1 (Node 2, i5-11600K, 64 GB DDR4)
   │
   ├── VLAN 10 Management: AdGuard, Step-CA, Suricata/Zeek NSM
   ├── VLAN 20 Servers:    nginx edge proxy, LGTM stack, Uptime Kuma, client LXCs
   ├── VLAN 30 AI_Compute: llama.cpp inference workloads
   ├── VLAN 40 Lab_Dev:    non-production experiments
   └── VLAN 50 IoT_Guest:  quarantined untrusted devices
```

### What is live in production today

| System | Status | Notes |
|---|---|---|
| Proxmox homelab, zero trust VLAN architecture | Live | OPNsense router on a stick, zero open inbound ports, Tailscale mesh for admin access |
| `spin-up-client.sh` provisioning pipeline | Live | Nine steps, fully automated, ~3 to 5 minutes zero to live |
| Portfolio (alfanotechport.org) | Live | HTTP/2 200, routed through the edge proxy and Cloudflare Tunnel |
| Business site (Alfano Tech) | Live | Booking form posts directly to Google Sheets CRM + Gmail notification pipeline |
| Prometheus + Node Exporter monitoring | Live | 9/9 targets up, auto-enrolled on new client containers |
| Grafana alerting (four rules) | Live | Container Down, CPU >85%, RAM >90%, Disk >80%, all SMTP routed |
| Cloudflare Tunnel (API driven) | Live | Tunnel ID managed remotely, all ingress rules injected via API |
| Cloudflare Worker failover per client | Live | Origin first, KV/Pages fallback when homelab is down, $0/mo, tested end to end |
| Client onboarding documents | Live | Web Services Agreement v3, Intake Form v2, Demo SOP v2, Onboarding SOP v2 |
| DNS delegation guide | Live | Click by click for GoDaddy, Namecheap, Google, Rebel.ca, Hover, IONOS |

### Provisioning pipeline, step by step

1. **Scripted LXC creation.** Unprivileged Debian 12 container, next free CTID from 204 up, VLAN 20, static IP in 10.20.20.0/24.
2. **Network wait.** Pings 1.1.1.1 until the container has a route.
3. **Package installation.** Apt installs nginx and node_exporter, verifies node_exporter is live on :9100.
4. **Vhost + placeholder.** Writes nginx server block and a placeholder index inside the client container.
5. **Reverse proxy registration.** Adds the client vhost on the edge proxy (CT 200).
6. **Cloudflare Tunnel ingress injection.** Fetches current tunnel config via the API, inserts the new ingress rule before the catch all, PUTs it back. Uses a temp file Python approach to avoid a known stdin conflict when heredocs consume curl piped input.
7. **DNS automation.** Auto-creates CNAME for `*.alfanotechport.org` subdomains. For external domains, prints a formatted manual delegation instruction block.
8. **Prometheus target enrollment.** Pulls prometheus.yml from CT 202, inserts the new scrape target, pushes it back, reloads Prometheus.
9. **Uptime Kuma enrollment.** Creates two monitors per client (ping + HTTP) via the Uptime Kuma API, with a monkey patch for the 2.2.1 `conditions` field bug.

End to end smoke test: curl through the edge proxy confirms HTTP 200 before the script declares success. No client site is considered live until that final probe passes.

---

## Project: Miner Control Center (MCC)

A high frequency desktop telemetry engine that unifies mixed fleets of 10,000+ ASICs into a single pane of glass.

- **Raw TCP sockets on port 4028** directly against miner firmware. Bypasses the slow HTTP admin interfaces that every off the shelf tool relies on.
- **Universal translator** normalizes unstructured string data from Antminers and JSON streams from Whatsminers into a single C# object model. One data shape, one UI, one alerting pipeline.
- **Thermal automation safety loop.** Asynchronous background watchdog polls chip level temperatures in real time. Any hashboard exceeding 85°C triggers an immediate Power Down packet to that specific unit, preventing catastrophic silicon failure before a human operator could even see the alert.
- **Stack:** C# .NET 8, WPF, SQLite, TCP/IP, regex heavy parsing.

---

## Technical Stack

| Domain | Tools |
|---|---|
| **Cloud & Infrastructure** | Azure (VM, VNet, NSG), Terraform (IaC), Cloudflare (Tunnel, Workers, Pages, KV, DNS, Zero Trust), Proxmox VE, LXC, Docker |
| **Observability** | Grafana, Loki, Tempo, Mimir, Prometheus, Node Exporter, Uptime Kuma, Suricata, Zeek |
| **Networking** | OPNsense, VLAN architecture, TCP/IP socket programming, packet analysis, firewall rule authoring, DPINGER multi WAN, Cat 5e through Cat 8 crimping, OTDR fiber diagnostics |
| **Languages** | C# (.NET 8), Python, Bash, PowerShell, SQL, JavaScript (vanilla, no frameworks) |
| **Web** | Hand coded HTML/CSS/JS, nginx, reverse proxy architecture, zero open port hosting |
| **AI / ML** | llama.cpp on CPU (NUMA aware pinning, thread tuning), MCP servers for LLM tool use, Claude Code for automation workflows |
| **Hardware** | BGA rework, microscope, Fluke multimeter, oscilloscope, hot air station, rail tracing, MOSFET and LDO diagnostics |
| **CI/CD (staged)** | Gitea, Drone CI, k3s, ArgoCD |

---

## Professional Experience

### Site Reliability Engineer
**SustainHash Technologies** · 2023 to Present

Own end to end SRE for five distributed compute facilities across Alberta. 300+ PH/s of managed compute capacity, from a remote edge site through a 200 PH/s flagship operation. Scope spans silicon to cloud: hardware, power, firmware, network, observability, and incident response. 24/7 on call. Executing dynamic load shedding and three phase power balancing through Alberta's full climate range.

### Professional Athlete (Defensive Line)
**Edmonton Elks, Canadian Football League** · 2024 to 2025

Selected in the 2024 CFL Supplemental Draft. Applied elite level discipline and team coordination in one of the highest pressure professional environments that exists. The work ethic transferred directly.

---

## Education

- **Lackawanna College** · Information Technology & Computer Science, 2022 to 2023
- **University of Colorado Boulder** · Integrative Physiology & Biological Sciences, 2020 to 2022
- **University of Alabama** · Kinesiology & Exercise Science, 2018 to 2020

Academic coursework focused on systems analysis, data patterns, and optimization metrics. The IT and CS coursework at Lackawanna specifically covered hardware architecture, IT systems administration, and cybersecurity fundamentals.

---

## Hard Won Lessons (Things That Cost Me Time So They Don't Cost You Time)

A short reference of non obvious failure modes I have personally hit in production and solved. If you are building on any of this stack, these are worth knowing.

**Cloudflare Tunnel is remotely managed.** When a tunnel is managed via the Zero Trust dashboard, the local `config.yml` on the cloudflared container is ignored entirely. All ingress automation must go through the Cloudflare API. GET the tunnel config, inject your rule before the catch all, PUT it back. Editing the local file will silently do nothing.

**A fully offline Proxmox container returns no data, not zero.** When a container goes completely down, Prometheus returns no data for the `up` metric rather than a value of 0. Grafana alert rules written as "IS BELOW 1" will therefore never fire on a dead container. The fix is to set "no data" handling to "Alerting" on every alert rule. Every SRE I know who runs Prometheus has been bitten by this at least once.

**stdin conflict between curl and python3 heredocs.** Piping curl output into `python3 -` with a heredoc body causes a stdin conflict. The heredoc consumes stdin, and Python receives nothing. The fix is to write the curl response to a temp file, write the Python logic to a separate temp file, and then execute them sequentially.

**Uptime Kuma 2.2.1 has an API conditions field bug.** Creating monitors programmatically in 2.2.1 fails in a non obvious way because of how the `conditions` field is serialized. Monkey patch required until upstream fixes it.

**Wrangler OAuth browser flow is unreliable on headless Windows dev machines.** Do not bother with the browser login. Authenticate via `CLOUDFLARE_API_TOKEN` environment variable only.

**AI site builders output React applications that require a Node runtime.** Lovable, Bolt.new, v0, and Replit all output full React apps. That is fundamentally incompatible with nginx static file hosting. They are useful for generating client facing mockups for approval. They are not a production output format.

**A single kinked SC/APC fiber patch cable can take down your entire production WAN.** I lost a production WAN to a physically kinked fiber in a poorly installed shared breaker/networking closet. The remediation playbook covered four layers: Telus technician engagement (G.657A2 fiber with proper bend tolerance, OPM readings, documented service loop), OPNsense multi WAN HA with DPINGER based failover, Cloudflare Tunnel as the IP shift resilient hosting layer that survives address changes, and physical layer remediation with fiber spiral wrap, wall mounted ONT, and a formal TIA-568 cable separation request to the building owner. Redundancy at one layer is not redundancy.

---

## Current Work

Active open items on the platform:

- **5G WAN failover integration.** GL.iNet Spitz AX X3000 modem and Intel I225-V 2.5GbE PCIe NIC queued to land. OPNsense tiered gateway groups will finalize once the hardware is in.
- **Gitea + ArgoCD GitOps loop.** k3s, Gitea, and ArgoCD are all running. Not yet wired into the client deployment workflow. Next priority after WAN redundancy closes.
- **OPNsense rule persistence.** Known issue where firewall rules require a shell reload after every reboot. Root cause investigation in progress.
- **Shared closet physical remediation.** Follow up with building owner on TIA-568 cable separation; implement fiber protection and strain relief as a permanent fix.

---

## Contact

- 🌐 **Portfolio:** [alfanotechport.org](https://alfanotechport.org)
- 💼 **LinkedIn:** [Antonio Alfano](https://linkedin.com/in/antonio-alfano-273162358)
- 📧 **Email:** [AntonioAlfano123456@gmail.com](mailto:AntonioAlfano123456@gmail.com)

![Antonio's GitHub Stats](https://github-readme-stats.vercel.app/api?username=AntonioDevTech&show_icons=true&theme=midnight-purple&hide_border=true&count_private=true)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=AntonioDevTech&layout=compact&theme=midnight-purple&hide_border=true)
