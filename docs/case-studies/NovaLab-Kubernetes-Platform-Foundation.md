# NovaLab Kubernetes Platform Foundation

Date: June 2026

## Objective

Build a repeatable Kubernetes platform on Proxmox using Infrastructure as Code, cloud-init, Helm, Metrics Server, cert-manager, and version-controlled engineering practices.

The goal was not simply to deploy Kubernetes. The goal was to construct a reusable platform foundation: infrastructure provisioning, operating-system configuration, administrator tooling, Kubernetes services, documentation, and repository structure that can support future platform evolution.

## Environment Summary

- Proxmox VE
- Ubuntu Server 24.04 LTS
- cloud-init
- Terraform
- K3s
- Helm
- kubectl
- k9s
- Metrics Server
- cert-manager
- Git and GitHub

## Completed Build

- Built a three-node K3s cluster.
- Deployed one control plane and two worker nodes.
- Provisioned virtual machines with Terraform.
- Used cloud-init for operating-system configuration.
- Configured persistent `KUBECONFIG`.
- Installed Helm and k9s for administrator workflows.
- Verified cluster health with `kubectl`.
- Installed Metrics Server and validated `kubectl top nodes` and `kubectl top pods`.
- Installed cert-manager using Helm.
- Verified the Helm release, CRDs, and running cert-manager pods.

## Architecture

```text
Proxmox Cluster
    |
    v
Terraform
    |
    v
Ubuntu Cloud Images
    |
    v
K3s Cluster
    - 1 control plane
    - 2 worker nodes
    - persistent KUBECONFIG
    - verified node and pod health
    |
    v
Helm
    |
    v
Platform Services
    - Metrics Server
    - cert-manager
    |
    v
Future Platform Components
    - MetalLB
    - Ingress
    - Prometheus
    - Grafana
    - Loki
    - Argo CD
    - Persistent storage
    - AI and media workloads
```

## Engineering Decision Log

| Decision | Reason | Outcome |
| --- | --- | --- |
| Keep Ubuntu clean | The base image should stay predictable and close to the vendor baseline. | Reduced drift and made rebuilds easier to reason about. |
| Automate administrator tooling with bootstrap scripts | Operator tooling changes more often than base OS provisioning. | Helm, k9s, and persistent kubeconfig setup can be repeated without baking everything into cloud-init. |
| Separate cloud-init from bootstrap scripts | cloud-init should handle initial OS identity, access, packages, and base configuration. | Responsibilities stayed clear instead of turning user-data into a catch-all script. |
| Use Terraform for infrastructure only | Terraform should create and shape infrastructure, not install every application inside Kubernetes. | Infrastructure state stayed separate from platform service management. |
| Use Helm for Kubernetes applications | Kubernetes services need release history, chart values, and repeatable install/upgrade behavior. | Metrics Server and cert-manager became managed platform components. |
| Organize the repository for growth | Platform work gains folders, scripts, docs, and decisions quickly. | The repo became easier to extend without losing operational context. |
| Favor reusable structure over one-off deployment | A platform foundation should survive the next service, cluster iteration, or rebuild. | The project moved from command learning to platform construction. |

## Operational Challenges

### Proxmox console behavior

Serial console behavior inside Proxmox was not always the right operational interface for Ubuntu cloud images. VGA console access was more useful during early troubleshooting because it made boot and login behavior easier to inspect.

Resolution: treat console type as part of VM template validation, not an afterthought. Use the console mode that supports the current troubleshooting need.

### SSH access and cloud-init

cloud-init disabled SSH password authentication, which is a better security default but can surprise you during first boot validation.

Resolution: rely on SSH key access and document the expected authentication path so a failed password login is not mistaken for a broken VM.

### Persistent KUBECONFIG

The cluster worked, but administrator access needed to survive new shell sessions. Without persistent `KUBECONFIG`, each session required extra manual setup.

Resolution: configure persistent kubeconfig handling in the administrator bootstrap flow.

### Helm installation

Helm was needed once the project moved from core cluster validation into platform services.

Resolution: install Helm as part of the administrator tooling layer, then use it for Kubernetes applications rather than folding those installs into Terraform.

### Kubernetes object hierarchy

cert-manager made the Kubernetes object model more visible. Installing it meant understanding Helm releases, namespaces, pods, services, deployments, and CRDs.

Resolution: verify each layer explicitly: Helm release, CRDs, namespace resources, and running pods.

## Lessons Learned

- Terraform creates infrastructure.
- cloud-init configures operating systems.
- Bootstrap scripts configure administrator tooling.
- Helm manages Kubernetes applications.
- CRDs extend Kubernetes with new object types.
- k9s provides an operational interface over `kubectl`.
- Repository organization matters early because platform projects grow quickly.

## Repository Improvements

- `README`: gives the project a clear entry point and current-state summary.
- `INSTALL` guide: preserves the build path so the platform can be repeated.
- `TROUBLESHOOTING` guide: captures operational issues and resolutions while they are still fresh.
- `bootstrap-control.sh`: automates administrator tooling setup and kubeconfig handling.
- `cloud-init` directory: keeps OS provisioning inputs separate from scripts and docs.
- `docs` directory: gives design notes, decisions, and runbooks a stable home.
- `scripts` directory: keeps repeatable operational commands out of one-off terminal history.

These changes matter because a platform repository becomes an operating artifact. The folder structure has to support future services, future troubleshooting, and future handoff.

## Foundation Readiness Assessment

| Area | Status |
| --- | --- |
| Infrastructure | Ready |
| Terraform | Ready |
| Cloud-init | Ready |
| Helm | Ready |
| Metrics | Ready |
| TLS Management | Ready |
| Ingress | Planned |
| Load Balancer | Planned |
| GitOps | Planned |
| Observability | Planned |
| Persistent Storage | Planned |
| Application Platform | Planned |

## Future Roadmap

| Phase | Focus | Components |
| --- | --- | --- |
| Phase 1 | Platform Foundation | Completed |
| Phase 2 | Networking | MetalLB, Ingress |
| Phase 3 | Platform Services | Prometheus, Grafana, Loki |
| Phase 4 | GitOps | Argo CD, automated deployments |
| Phase 5 | AI and Media Platform | GPU workloads, inference, Tdarr, Jellyfin |

## Matt's Notes

What surprised me most was how quickly Kubernetes stopped being about commands and started being about boundaries. Terraform, cloud-init, bootstrap scripts, Helm, and Kubernetes each have a job. The platform gets easier to understand when each layer is allowed to do its job.

The biggest lesson was that Kubernetes is not just a thing you install. It is an operational surface. Once Metrics Server and cert-manager were installed, I had to think in terms of releases, pods, namespaces, CRDs, health checks, and upgrade paths.

This also changed how I understood my own automation history. The same philosophy I used years ago with Windows batch deployment scripts still applies here: remove repeatable manual steps, make the next run cleaner, and leave enough documentation that the process can be trusted later.

Repository organization mattered earlier than expected. At first, folders feel like ceremony. Then the project gains cloud-init files, scripts, guides, troubleshooting notes, and future roadmap items. Structure becomes part of the platform.

This project marks a real transition for me: from learning Kubernetes commands to building a reusable platform foundation that can grow into networking, observability, GitOps, storage, and eventually AI workloads.
