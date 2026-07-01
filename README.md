# Antonio Alfano

**Site Reliability Engineer · AI Platform Engineer · Hardware Down to the Silicon**

Edmonton, Alberta · Dual 🇺🇸 / 🇨🇦 Citizen · No sponsorship required

[![Portfolio](https://img.shields.io/badge/Portfolio-alfanotechport.org-0A66C2?style=flat-square)](https://alfanotechport.org)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Antonio_Alfano-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/antonio-alfano-273162358)
[![Infrastructure](https://img.shields.io/badge/Infrastructure-Live_Production-22C55E?style=flat-square)]()
[![SLA](https://img.shields.io/badge/SLA_Target-99.9%25-22C55E?style=flat-square)]()
[![AI](https://img.shields.io/badge/AI-Local_First_Inference-8B5CF6?style=flat-square)]()

---

I build systems that span the entire stack — from BGA rework on ASIC hashboards under a microscope to a local-first, 7-layer compound-AI pipeline I designed and run on my own hardware. Most AI engineers can't diagnose their infrastructure at Layer 1. Most SREs can't build an inference pipeline with routing, retrieval, verification, and audit as first-class stages. I do both, in production, and I own the pager.

I currently run end-to-end Site Reliability Engineering for five distributed compute facilities across Alberta totaling **300+ PH/s** of managed capacity, and I operate a self-hosted multi-tenant hosting platform off a Proxmox homelab I designed, built, and automated from bare metal up. I do not use drag-and-drop site builders, I do not resell AWS, and I do not ship infrastructure I cannot trace to the silicon.

---

## By the Numbers

| | |
|---|---|
| **5** distributed compute sites | **300+ PH/s** managed capacity |
| **Zero** open inbound ports, always | **~3–5 min** zero-to-live client provisioning (9 automated steps) |
| **7-layer** compound-AI pipeline, self-hosted | **4** local model roles, hard no-CPU-spill contract |
| **10,000+** ASICs unified in one telemetry pane | **15-second** WAN failover, validated in a real Layer 1 fiber outage |

---

## Flagship: Nabu — Local-First Compound AI Operations System

A private, local-first AI system built on the principle of **one brain, many shells**: a single reasoning layer, durable context stored *outside* the models, surfaced through multiple interfaces. It runs entirely on personal homelab hardware with no external API dependency for inference. The design goal is an AI teammate that reads real evidence, verifies its own answers, and admits when it doesn't know — instead of confidently hallucinating.

Every request flows through a 7-stage pipeline, not a single prompt:

```
Request
  │
  ▼
[1] Input validation ────── reject malformed / unsafe input at the door
  │
  ▼
[2] Router / intent ─────── qwen3:4b classifies intent + confidence
  │
  ▼
[3] Evidence gather (RAG) ─ LanceDB hybrid search · bge-m3 embeddings · bge-reranker
  │
  ▼
[4] Context compression ─── fit retrieved evidence to a hard token budget
  │
  ▼
[5] Generation ─────────── role-specific local model synthesizes the draft
  │
  ▼
[6] Verification ────────── answer is checked against the retrieved evidence
  │
  ▼
[7] Audit ──────────────── append-only event log · full per-turn trace
  │
  ▼
Response  (honest-miss when evidence is insufficient — never fabricated)
```

**One model role at a time, by design.** A hard no-CPU-spill rule keeps every model fully resident on the GPU. Roles are swapped deterministically rather than run concurrently, so inference never degrades into swap thrash.

| Role | Model | Job |
|---|---|---|
| Router | `qwen3:4b` | Intent and confidence classification |
| Coder | `qwen2.5-coder:7b` | Code generation and analysis |
| Finalizer / Draft | `qwen3:8b` | Synthesis and drafting |
| Vision | `qwen3-vl:8b` | Image and screen understanding |

**Stack:** Python / FastAPI backend · React + TypeScript control center · Tauri desktop overlay · Ollama for local model serving · LanceDB for hybrid vector + keyword retrieval · SQLite event store.

**Engineering discipline is the actual point.** This is a solo build held to production standards:

- **Enforced module budgets.** A budget gate fails any file that exceeds its line ceiling (warn at 700, hard stop at 1,200). Logic is not allowed to metastasize into a single god-file.
- **Self-healing with a human-in-the-loop ceiling.** Routine, measurable fixes auto-apply. Anything touching the reasoning core, stored memory, or safety machinery is gated behind explicit human approval. The safety layer is an absolute carve-out that is never auto-edited.
- **Append-only remediation ledger.** Every decision, error pattern, and change is logged. Fixes land one at a time, through a real pre-commit gate, with no bypass.
- **~900 automated tests**, plus a standing rule that the test suite is *not* trusted as the acceptance oracle — cold-start behavior against the real inference path is. Mocked GPU tests produce false-green results, so proof happens on the live human path.

The architecture draws directly from current research on compound AI systems, self-verification, and skill optimization. It is treated as a serious engineering platform, not a chatbot wrapper.

---

## Flagship: Project Titan — Automated Multi-Tenant Hosting Platform

A fully automated, self-hosted, multi-tenant web hosting platform on a Proxmox homelab. Every design decision optimizes for availability per dollar and near-zero operational overhead.

```
Internet
   │
   ▼
Cloudflare (Tunnel · Workers · DNS · SSL · Pages failover)
   │
   ▼  ingress only — zero open inbound ports
OPNsense firewall (router-on-a-stick)
   │
   ▼  VLAN tagged
Proxmox VE
   │
   ├── VLAN 10 Management:  AdGuard · Step-CA · Suricata/Zeek NSM
   ├── VLAN 20 Servers:     nginx edge proxy · LGTM stack · Uptime Kuma · client LXCs
   ├── VLAN 30 AI_Compute:  local inference workloads
   ├── VLAN 40 Lab_Dev:     non-production experiments
   └── VLAN 50 IoT_Guest:   quarantined untrusted devices
```

**Provisioning pipeline — zero to live, nine automated steps, no dashboard clicks:**

1. **Scripted LXC creation** — unprivileged Debian container, next free CTID, VLAN 20, static IP.
2. **Network wait** — poll until the container has a route.
3. **Package install** — nginx and node_exporter, verify the exporter is live on `:9100`.
4. **Vhost + placeholder** — nginx server block and placeholder index inside the container.
5. **Reverse proxy registration** — add the client vhost on the edge proxy.
6. **Cloudflare Tunnel ingress injection** — GET the tunnel config via API, insert the rule before the catch-all, PUT it back.
7. **DNS automation** — auto-create the CNAME for internal subdomains; print a formatted delegation block for external domains.
8. **Prometheus target enrollment** — pull, patch, push, reload — the new scrape target auto-registers.
9. **Uptime Kuma enrollment** — two monitors per client (ping + HTTP) via API.

No site is declared live until a final end-to-end smoke test returns HTTP 200 through the edge proxy.

**Live in production today:**

| System | Status | Notes |
|---|---|---|
| Zero-trust VLAN architecture | Live | Router-on-a-stick, zero open inbound ports, Tailscale mesh for admin |
| `spin-up-client.sh` pipeline | Live | Nine steps, ~3–5 min zero-to-live |
| LGTM observability stack | Live | Grafana · Loki · Tempo · Mimir · Prometheus · Node Exporter |
| Prometheus monitoring | Live | Targets auto-enroll on every new client container |
| Grafana alerting | Live | Container Down · CPU · RAM · Disk — all SMTP-routed, all handling the no-data case |
| Cloudflare Tunnel (API-driven) | Live | All ingress rules injected via API, never local YAML |
| Cloudflare Worker failover | Live | Origin-first, KV/Pages fallback when the homelab is down — $0/mo, tested end to end |
| Dual-WAN failover | Live | Telus fiber primary, cellular secondary — different carrier by design |

---

## Flagship: Miner Control Center — High-Frequency Fleet Telemetry

A desktop telemetry engine that unifies mixed fleets of **10,000+ ASICs** into a single pane of glass.

- **Raw TCP sockets on port 4028**, straight against miner firmware — bypassing the slow HTTP admin interfaces every off-the-shelf tool depends on.
- **Universal translator** normalizes unstructured Antminer strings and Whatsminer JSON into one C# object model. One data shape, one UI, one alerting pipeline.
- **Thermal safety loop** — an async watchdog polls chip-level temperatures in real time. Any hashboard over **85°C** triggers an immediate power-down packet to that specific unit, before a human operator could even see the alert.
- **Stack:** C# .NET 8 · WPF · SQLite · TCP/IP · regex-heavy parsing.

---

## Technical Stack

| Domain | Tools |
|---|---|
| **AI / ML** | Local-first inference (Ollama, llama.cpp) · compound-AI pipeline design · RAG with LanceDB hybrid search · bge-m3 embeddings + bge-reranker · NUMA-aware CPU pinning and thread tuning · MCP servers for LLM tool use · self-verification and audit layers |
| **Automation** | End-to-end provisioning pipelines · API-driven infrastructure (Cloudflare, Uptime Kuma, Prometheus) · self-healing systems with human-in-the-loop approval gates · Bash · Python · PowerShell |
| **Cloud & Infra** | Azure (VM, VNet, NSG) · Terraform · Cloudflare (Tunnel, Workers, Pages, KV, DNS, Zero Trust) · Proxmox VE · LXC · Docker |
| **Observability** | Grafana · Loki · Tempo · Mimir · Prometheus · Node Exporter · Uptime Kuma · Suricata · Zeek |
| **Networking** | OPNsense · VLAN architecture · TCP/IP socket programming · packet analysis · firewall rule authoring · multi-WAN failover · Cat 5e–Cat 8 crimping · OTDR fiber diagnostics |
| **Languages** | C# (.NET 8) · Python · Bash · PowerShell · SQL · TypeScript · JavaScript |
| **Web** | React + TypeScript · Tauri · hand-coded HTML/CSS/JS · nginx · reverse-proxy architecture · zero-open-port hosting |
| **Hardware** | BGA rework · microscope · Fluke multimeter · oscilloscope · hot air station · voltage-rail tracing · MOSFET and LDO diagnostics |
| **CI/CD** | Gitea · Drone CI · k3s · ArgoCD · pre-commit gate enforcement |

---

## Hard-Won Lessons

Non-obvious failure modes I have personally hit in production and solved. If you're building on any of this stack, these are worth the read.

**Cloudflare Tunnel is remotely managed.** When a tunnel is managed via the Zero Trust dashboard, the local `config.yml` is ignored entirely. All ingress automation must go through the API — GET the config, inject your rule before the catch-all, PUT it back. Editing the local file silently does nothing.

**A fully offline container returns no data, not zero.** When a container goes completely down, Prometheus returns *no data* for the `up` metric rather than `0`. Alert rules written as "IS BELOW 1" will therefore never fire on a dead container. The fix is to set no-data handling to "Alerting" on every rule. Every Prometheus SRE gets bitten by this once.

**stdin conflict between curl and python3 heredocs.** Piping curl output into `python3 -` with a heredoc body causes a stdin conflict — the heredoc consumes stdin and Python receives nothing. Write the curl response and the Python logic to separate temp files, then execute sequentially.

**Never trust a mocked test suite for live AI behavior.** A suite that mocks the GPU/inference path produces false-green results. The only valid acceptance oracle is the real human path with a cold model. This lesson has to be re-enforced at the start of every session.

**Substring keyword routing is a recurring defect class.** Matching "path" inside "best-path," or "api" inside "capital," silently misroutes intent. Every routing fix has to check for substring collision, not just exact match.

**A single kinked fiber patch cable can take down your entire production WAN.** I lost a production WAN to a physically kinked SC/APC fiber in a poorly installed shared closet. Remediation spanned four layers: carrier engagement with proper bend-tolerant fiber, OPNsense multi-WAN HA with DPINGER failover, Cloudflare Tunnel as the IP-shift-resilient hosting layer, and physical strain relief plus a formal TIA-568 cable-separation request. Redundancy at one layer is not redundancy.

**AI site builders output React apps that need a Node runtime.** Lovable, Bolt.new, v0, and Replit all emit full React applications — fundamentally incompatible with nginx static hosting. Useful for client-facing mockups, not a production output format.

---

## Experience

**Site Reliability Engineer** — SustainHash Technologies · 2023–Present
Own end-to-end SRE for five distributed compute facilities across Alberta — 300+ PH/s of managed capacity, from a remote edge site to a 200 PH/s flagship. Scope spans silicon to cloud: hardware, power, firmware, network, observability, and incident response. 24/7 on call. Dynamic load shedding and three-phase power balancing across Alberta's full climate range, from 30 below to 95 above.

**Professional Athlete, Defensive Line** — Edmonton Elks, CFL · 2024–2025
Selected in the 2024 CFL Supplemental Draft. Elite-level discipline and team coordination under real pressure. The work ethic transferred directly.

---

## Education

- **Lackawanna College** — Information Technology & Computer Science · 2022–2023
- **University of Colorado Boulder** — Integrative Physiology & Biological Sciences · 2020–2022
- **University of Alabama** — Kinesiology & Exercise Science · 2018–2020

Coursework centered on systems analysis, data patterns, and optimization metrics, with hardware architecture, systems administration, and cybersecurity fundamentals at Lackawanna.

---

## Contact

- 🌐 **Portfolio:** [alfanotechport.org](https://alfanotechport.org)
- 💼 **LinkedIn:** [Antonio Alfano](https://linkedin.com/in/antonio-alfano-273162358)
- 📧 **Email:** [AntonioAlfano123456@gmail.com](mailto:AntonioAlfano123456@gmail.com)

![Antonio's GitHub Stats](https://github-readme-stats.vercel.app/api?username=AntonioDevTech&show_icons=true&theme=midnight-purple&hide_border=true&count_private=true)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=AntonioDevTech&layout=compact&theme=midnight-purple&hide_border=true)
