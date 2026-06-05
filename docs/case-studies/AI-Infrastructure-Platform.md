NOVALAB AI INFRASTRUCTURE PROJECT RECAP
Date: June 2026

=========================================================
AI TRAINING ROADMAP
=========================================================

Phase 0 - Local AI Infrastructure Platform (COMPLETED)

Phase 1 - AI Infrastructure Fundamentals
- Models
- Tokens
- Context Windows
- Quantization
- GPU Inference
- VRAM Planning

Phase 2 - llama.cpp
- GGUF Models
- Quantization
- CUDA Builds
- Hugging Face Models
- Inference Optimization

Phase 3 - vLLM
- Production Model Serving
- OpenAI Compatible APIs
- High Performance Inference
- Enterprise AI Architecture

Phase 4 - Vector Databases
- Qdrant
- ChromaDB
- Embeddings
- Semantic Search

Phase 5 - RAG
- Retrieval Augmented Generation
- Document Ingestion
- Chunking
- Knowledge Retrieval

Phase 6 - Secret Management
- Vaultwarden
- HashiCorp Vault
- Secrets Management
- Certificate Management

Phase 7 - AI Operations Agents
- MCP
- Tool Calling
- Automation
- Workflow Agents

Phase 8 - AI Platform Engineering
- Kubernetes
- GPU Scheduling
- GitOps
- Observability
- AI Operations

=========================================================
PROJECT DOCUMENTATION STANDARD
=========================================================

Every project must contain:

- Objective
- Executive Summary
- Environment
- Architecture
- Architecture Diagram
- Technologies Used
- Secret Management
- Implementation
- Challenges Encountered
- Root Cause Analysis
- Resolution
- Lessons Learned
- Observability
- Backup & Recovery
- Security Review
- Business Value
- Portfolio Potential
- Future Enhancements
- Interview Story

=========================================================
LEARNING JOURNAL STANDARD
=========================================================

Date
Training Track
Time Spent
Topic
Objective
What Was Learned
What Was Built
What Broke
Root Cause
Resolution
Lessons Learned
Portfolio Potential
Interview Story Potential
GitHub Repository
Related Project
Next Steps

=========================================================
INTERVIEW STORY FORMAT
=========================================================

Situation
Task
Action
Result
Lesson Learned

Every future project should produce an interview story.

=========================================================
PROJECT BACKLOG
=========================================================

Completed
- Local AI Infrastructure Platform

Planned
- llama.cpp
- vLLM
- Vector Database Platform
- RAG Knowledge Platform
- Secret Management Platform
- AI Operations Agent
- AI Platform Engineering Lab

=========================================================
PROJECT #1
=========================================================

Title:
Building a Local AI Infrastructure Platform

Technologies:
- Proxmox
- RTX 3090
- NVIDIA 595.80
- CUDA 13.2
- Open WebUI
- Ollama
- Qwen 7B
- Qwen 14B
- GPT-OSS 20B

Major Challenges:
- NVIDIA Driver Installation
- GPU Ownership Conflict
- NovaCore Conflict
- Ollama Validation
- Platform Trust Issues

Root Cause:
NovaCore conflict prevented NVIDIA driver from correctly attaching to GPU.

Resolution:
- Rebuilt Proxmox
- Established clean baseline
- Installed NVIDIA 595.80
- Validated CUDA
- Validated Ollama
- Validated Open WebUI
- Validated GPU Inference

Lessons Learned:
- Trusting a clean baseline is often more valuable than endlessly troubleshooting an unknown environment.
- Symptoms are not always root causes.
- Documentation matters.
- Recovery must be part of the build.
- AI Infrastructure is still Infrastructure.

Business Value:
- GPU Infrastructure
- AI Infrastructure
- Linux Administration
- Troubleshooting
- Platform Engineering
- Backup Strategy
- Documentation

Future Enhancements:
- llama.cpp
- vLLM
- Vector Database
- RAG
- AI Operations Agent

=========================================================
WEBSITE STATUS
=========================================================

Completed:
- Homepage
- Current Mission
- Infrastructure Philosophy
- Strategic Development Areas
- NovaLab Status
- AI Lab Section
- Projects Section
- Case Study Framework

Domain:
matthewsmalling.com

GitHub:
ai-portfolio-website

Dedicated Project Page:
projects/building-local-ai-infrastructure-platform.html

=========================================================
NEW DISCOVERY
=========================================================

AI Platform #1

Proxmox
RTX 3090
Linux
Open WebUI
Ollama
GPT-OSS 20B

AI Platform #2

Windows 11
Ryzen 9 9950X3D
AMD Radeon 7900 XTX
Open WebUI
Ollama
GPT-OSS 20B

Validation Results:

Qwen 7B:
100% GPU

Qwen 14B:
100% GPU

GPT-OSS 20B:
100% GPU

Conclusion:
7900 XTX is a legitimate local AI platform and not merely a gaming card.

Future Article:
RTX 3090 vs AMD 7900 XTX
Local AI Infrastructure Comparison

=========================================================
CORE RULE GOING FORWARD
=========================================================

Learn
↓
Build
↓
Document
↓
GitHub
↓
Website
↓
Interview Story

No more:

Learn
↓
Forget

=========================================================
CURRENT TARGET ROLES
=========================================================

Near-Term:
- Director of Infrastructure
- Senior Director of Infrastructure
- Head of Infrastructure
- Director of Platform Engineering

Mid-Term:
- Director of Infrastructure & AI Operations
- AI Infrastructure Manager
- Head of Platform Engineering

Long-Term:
- Director of AI Infrastructure
- VP Infrastructure & AI Platforms
- CTO

=========================================================
NEXT STEP
=========================================================

Project #2:
llama.cpp

Then:
vLLM

Then:
Vector Databases

Then:
RAG

Then:
AI Operations Agents

Then:
AI Platform Engineering

=========================================================
Chron0s-01 GPS Time Server Rebuild
=========================================================

Project Goal
---------------------------------------------------------

Chron0s-01 is a Raspberry Pi 5 based GPS/GNSS time server intended to provide reliable local NTP service for the homelab.

The build uses:
- chrony
- gpsd
- gpsd-clients
- pps-tools

The goal is simple: keep local infrastructure time sync available even when relying less on outside services, and learn the actual validation path instead of treating NTP like magic.

Hardware Used
---------------------------------------------------------

- Raspberry Pi 5
- NVMe boot drive
- Pi 5 NVMe HAT/ribbon
- USB GPS/GNSS receiver: u-blox M10 GR-U01
- USB-C power supply and known-good USB-C cable
- WiFi temporarily used during setup, with wired Ethernet planned later

Build Notes and Troubleshooting
---------------------------------------------------------

- Initial boot worked, but the first Pi showed EXT4 filesystem issues, input/output errors, and commands disappearing.
- The NVMe SMART report showed the Lexar NM790 was healthy: SMART passed, Critical Warning 0x0, Media and Data Integrity Errors 0, Error Information Log Entries 0.
- Reseating the Pi 5 PCIe/NVMe ribbon helped temporarily.
- Moving the NVMe/HAT stack to a second Raspberry Pi 5 and switching to a known-good USB-C cable resulted in stable operation.
- The first Pi/cable combination had shown throttled=0x50000, meaning historical undervoltage/throttling.
- The second Pi showed throttled=0x0, meaning no undervoltage or throttling events.
- Working theory: the failures were most likely caused by power/cable instability or Pi #1 hardware behavior, not the NVMe drive.

Useful Validation Commands
---------------------------------------------------------

```bash
vcgencmd get_throttled
vcgencmd measure_temp
mount | grep " / "
dmesg | grep -i nvme
dmesg | grep -iE "error|i/o|ext4"
sudo smartctl -a /dev/nvme0
chronyd -v
gpsd -V
cgps -s
gpspipe -r | head
gpspipe -w -n 20
ls -l /dev/pps*
sudo ppstest /dev/pps0
chronyc sources -v
chronyc tracking
```

GPS/GNSS Validation
---------------------------------------------------------

The GR-U01 appeared as:
- /dev/ttyUSB0
- Prolific PL2303 USB serial bridge
- u-blox M10 receiver detected by gpsd

GPS achieved a 3D fix near a window. cgps showed satellite count, latitude/longitude, altitude, and 3D FIX. gpspipe confirmed live NMEA/JSON output from gpsd.

PPS Status
---------------------------------------------------------

- /dev/pps0 existed.
- ppstest found the PPS source but timed out waiting for pulses.
- gpsd reported pps:false.
- Current status: GPS time works; PPS device exists but PPS pulse is not yet validated.
- PPS investigation is deferred to a later phase.

Current Status
---------------------------------------------------------

Status: Chron0s-01 base platform is stable on the second Raspberry Pi 5. WiFi, SSH, NVMe boot, chrony, gpsd, GPS lock, and satellite monitoring are working. PPS remains unresolved.

Next Steps
---------------------------------------------------------

- Let cgps -s run for several hours to monitor satellite stability.
- Periodically check vcgencmd get_throttled.
- Configure chrony to consume GPS time from gpsd.
- Revisit PPS troubleshooting later.
- Move to wired Ethernet/static IP when convenient.
- Replace the oversized 1TB NM790 with the incoming 128GB NVMe for a permanent low-maintenance build.
