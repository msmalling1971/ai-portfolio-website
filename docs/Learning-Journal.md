# NovaLab Learning Journal

## Entry Template

### Date

### Training Track

* AI Infrastructure
* Platform Engineering
* Terraform
* Kubernetes
* Security
* Other

### Time Spent

### Topic

### Objective

What was I trying to learn?

### What Was Learned

### What Was Built

### What Broke

### Root Cause

### Resolution

### Lessons Learned

### Portfolio Potential

Yes / No

### Interview Story Potential

Yes / No

### GitHub Repository

### Related Project

### Next Steps


# Entries

## June 1, 2026

### Training Track
AI Infrastructure

### Time Spent
8+ hours

### Topic
Local AI Infrastructure Platform

### Objective
Build a stable local AI platform using Proxmox, RTX 3090, Open WebUI, Ollama, and GPT-OSS 20B.

### What Was Learned
- NVIDIA driver installation
- GPU inference
- Ollama model management
- Open WebUI configuration
- Backup strategy

### What Was Built
- Working AI server
- Open WebUI
- Ollama
- GPT-OSS 20B
- GitHub portfolio repository

### What Broke
- NVIDIA driver installation
- NovaCore driver conflict
- GPU binding issues

### Root Cause
Conflicting driver/module ownership prevented NVIDIA from attaching correctly.

### Resolution
Rebuilt Proxmox and installed a clean NVIDIA stack.

### Lessons Learned
Trusting the baseline is often more important than endlessly troubleshooting symptoms.

### Portfolio Potential
Yes

### Interview Story Potential
Yes

### Related Project
Local AI Infrastructure Platform

### Next Steps
- Create case study
- Review llama.cpp
- Review vLLM