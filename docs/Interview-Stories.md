# Building a Local AI Infrastructure Platform

## Executive Summary

This project focused on building a local AI infrastructure platform capable of hosting and serving modern large language models using Proxmox, NVIDIA GPU acceleration, Open WebUI, and Ollama.

The objective was not simply to run AI models, but to create a repeatable platform for AI experimentation, infrastructure learning, documentation assistance, future RAG development, and future AI agent development.

The final environment successfully hosted and executed Qwen 7B, Qwen 14B, and GPT-OSS 20B models using GPU acceleration while incorporating backup, recovery, and platform management practices.

The project also provided hands-on experience with GPU infrastructure, Linux administration, AI runtime platforms, troubleshooting complex driver conflicts, and platform engineering concepts that will support future AI infrastructure initiatives.

---

## Objective

Build a stable local AI platform capable of:

* Hosting modern large language models
* Leveraging GPU acceleration
* Supporting future RAG implementations
* Supporting future AI agents and automation
* Providing a controlled environment for AI infrastructure learning
* Maintaining local control over data and experimentation

The goal was to learn AI infrastructure from an operations and platform engineering perspective rather than solely from a model-consumption perspective.

## Environment

### Hardware

#### AI Server

* Proxmox Host
* NVIDIA GeForce RTX 3090 (24GB VRAM)
* Local NAS storage for backups
* Dedicated AI infrastructure environment

#### Secondary Validation Platform

* AMD Ryzen 9 9950X3D
* AMD Radeon 7900 XTX (24GB VRAM)
* Windows 11
* Ollama and Open WebUI validation environment

### Software Stack

#### Virtualization Layer

* Proxmox VE

#### GPU Layer

* NVIDIA Driver 595.80
* CUDA 13.2

#### AI Runtime Layer

* Ollama

#### User Interface Layer

* Open WebUI

#### Models Validated

* Qwen 2.5 7B
* Qwen 3 14B
* GPT-OSS 20B

### Storage and Recovery

* Proxmox Snapshots
* NAS-Based Backup Repository
* GitHub Documentation Repository

---

## High-Level Architecture

### Logical Architecture

Proxmox Host
↓
RTX 3090 GPU
↓
NVIDIA Driver Stack
↓
CUDA Runtime
↓
LXC AI Container
↓
Open WebUI
↓
Ollama
↓
Large Language Models

Models Tested

* Qwen 2.5 7B
* Qwen 3 14B
* GPT-OSS 20B

### Platform Purpose

The platform serves as:

* AI Learning Environment
* Infrastructure Research Platform
* Future RAG Platform
* Future Agent Platform
* Future Documentation Assistant
* Future Knowledge Management System

### Architecture Diagram

(To be replaced with draw.io diagram in a future revision)

````

---

## Challenges Encountered

### Challenge 1: Unstable AI Platform Behavior

Initial testing produced inconsistent behavior when attempting to run GPU-accelerated AI workloads.

Symptoms included:

* Model startup failures
* Slow responses
* GPU utilization inconsistencies
* Ollama runner failures
* Unexpected inference behavior

The platform appeared functional but did not consistently perform as expected.

---

### Challenge 2: NVIDIA Driver Installation Failures

During platform validation, NVIDIA driver installation repeatedly failed.

Symptoms included:

* Driver installation errors
* NVIDIA modules failing to load
* Missing GPU functionality
* Inconsistent AI runtime behavior

The issue initially appeared to be related to driver installation but was later determined to be a deeper platform-level conflict.

---

### Challenge 3: GPU Ownership Conflict

Investigation revealed that the RTX 3090 was being claimed by an unexpected kernel-level component.

This prevented the NVIDIA driver from properly attaching to the GPU.

As a result:

* CUDA functionality was impaired
* AI inference could not be reliably validated
* Troubleshooting efforts became increasingly difficult due to uncertainty regarding platform state

---

### Challenge 4: Trust In The Baseline

A key challenge throughout the project was determining whether the environment itself could be trusted.

Multiple troubleshooting paths were explored.

Eventually the decision was made to rebuild the environment from a known baseline rather than continue troubleshooting an increasingly complex configuration.

This proved to be a critical decision in the overall success of the project.

---

## Root Cause Analysis

### Primary Root Cause

The NVIDIA GPU was ultimately discovered to be associated with a conflicting driver stack involving NovaCore.

The conflict prevented the NVIDIA driver from correctly attaching to the RTX 3090.

As a result:

* GPU ownership was ambiguous
* Driver installation validation failed
* AI workloads could not be reliably executed

---

### Secondary Root Cause

The environment had accumulated enough uncertainty that troubleshooting confidence began to decrease.

Without a trusted baseline, it became difficult to determine whether new problems were caused by:

* Current configuration
* Historical changes
* Driver conflicts
* Platform state drift

The rebuild eliminated these unknowns.

---

## Resolution

The final resolution involved:

### Platform Rebuild

* Fresh Proxmox deployment
* Clean operating environment
* Updated host platform

### GPU Validation

* Removal of conflicting GPU ownership
* NVIDIA driver installation
* CUDA validation
* GPU visibility verification

### AI Platform Validation

* Open WebUI validation
* Ollama validation
* Model deployment testing

### Model Verification

Successful execution of:

* Qwen 2.5 7B
* Qwen 3 14B
* GPT-OSS 20B

using GPU acceleration.

### Recovery Strategy

After successful validation:

* Snapshots were created
* NAS backups were created
* GitHub documentation was updated

to establish a recoverable baseline.
## Lessons Learned

### Lesson 1: Trust The Baseline

One of the most important lessons from this project was the value of a trusted baseline.

As troubleshooting complexity increased, confidence in the environment decreased.

The eventual decision to rebuild the platform from a clean starting point significantly accelerated progress and reduced uncertainty.

A clean baseline often provides more value than endless troubleshooting against an unknown state.

---

### Lesson 2: Symptoms Are Not Always Root Causes

Initial troubleshooting focused on:

* Ollama
* GPU utilization
* Model execution

The true root cause ultimately involved GPU ownership and driver conflicts.

This reinforced the importance of validating assumptions and investigating dependencies rather than focusing exclusively on visible symptoms.

---

### Lesson 3: Documentation Matters

Throughout the project, maintaining notes, screenshots, configuration details, and lessons learned improved troubleshooting efficiency and reduced repeated work.

Documentation transformed the project from an experiment into a repeatable process.

---

### Lesson 4: Recovery Must Be Part Of The Build

Snapshots and backups were not treated as an afterthought.

A stable platform is only valuable if it can be recovered.

Establishing snapshot and backup procedures immediately after validation created a reliable foundation for future experimentation.

---

### Lesson 5: AI Infrastructure Is Infrastructure

Running AI workloads successfully requires the same operational disciplines as any other production platform:

* Architecture
* Reliability
* Security
* Monitoring
* Recovery
* Documentation

The technology changes, but the operational principles remain the same.

---

## Business Value

This project demonstrates practical experience with:

### Infrastructure Operations

* Linux administration
* Platform troubleshooting
* Environment validation

### GPU Infrastructure

* NVIDIA driver management
* CUDA runtime validation
* GPU workload execution

### AI Infrastructure

* Open WebUI
* Ollama
* Model deployment
* Local inference

### Platform Engineering Concepts

* Repeatable deployments
* Documentation
* Version control
* Operational procedures

### Disaster Recovery

* Snapshots
* Backups
* Recoverability
* Baseline management

### Decision Making

The project also demonstrates the ability to determine when rebuilding a platform is more effective than continuing to troubleshoot an increasingly uncertain environment.

---

## Portfolio Potential

### Website Case Study

Yes

### GitHub Documentation

Yes

### LinkedIn Article Potential

High

### Interview Story Potential

High

---

## Interview Story Summary

### Situation

A local AI infrastructure platform was required to support AI learning, experimentation, and future platform engineering initiatives.

### Task

Build a reliable GPU-accelerated AI environment using Proxmox, Open WebUI, Ollama, and modern language models.

### Action

* Diagnosed platform issues
* Investigated GPU ownership conflicts
* Rebuilt the environment
* Validated NVIDIA and CUDA functionality
* Verified model execution
* Implemented backup and recovery procedures

### Result

Successfully deployed and validated:

* Qwen 2.5 7B
* Qwen 3 14B
* GPT-OSS 20B

with full GPU acceleration.

### Lesson

Trusting a clean baseline can be more valuable than endlessly troubleshooting an unknown environment.

---

## Future Enhancements

### Near Term

* llama.cpp
* vLLM
* Model benchmarking

### Mid Term

* Vector Database
* RAG Platform
* Knowledge Assistant

### Long Term

* AI Operations Agent
* AI Platform Engineering Environment
* Kubernetes-Based AI Infrastructure

---

## Executive Summary

This project successfully established a local AI infrastructure platform capable of supporting modern GPU-accelerated language models.

The effort required troubleshooting driver conflicts, validating GPU ownership, rebuilding the platform from a trusted baseline, and implementing operational controls including backup and recovery.

The final result is a repeatable AI infrastructure environment that serves as the foundation for future work involving RAG, vector databases, AI agents, and platform engineering initiatives.
