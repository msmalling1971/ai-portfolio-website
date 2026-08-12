# Genesis Kubernetes Platform Foundation

Date: August 2026

## Objective

Build and validate an enterprise-style Kubernetes development platform on Proxmox with highly available API access, layered disaster recovery, off-node S3-compatible protection, and explicit storage boundaries.

The goal is not simply to deploy Kubernetes. Genesis is a working private-lab platform and a hands-on architecture project that makes implementation, compatibility assessment, failure modes, recovery, and operational policy testable.

## Environment Summary

- Proxmox VE
- Ubuntu Server 24.04 LTS
- cloud-init
- Terraform
- K3s
- Three control-plane/server nodes plus worker nodes
- HAProxy and Keepalived
- Kubernetes API VIP `192.168.70.5`
- Helm
- kubectl
- k9s
- Metrics Server
- cert-manager
- Headlamp and Kubernetes RBAC
- MetalLB installed/worked with as a service-exposure option
- K3s embedded-etcd snapshots
- Velero v1.18.2 with AWS/S3 plugin v1.14.2
- Local S3-compatible RustFS object storage
- Git and GitHub

## Completed Build

- Built a K3s cluster with three control-plane/server nodes plus worker nodes.
- Implemented a highly available Kubernetes API endpoint at `192.168.70.5` using two HAProxy and Keepalived VMs.
- Deliberately tested and validated VIP failover.
- Provisioned virtual machines with Terraform.
- Used cloud-init for operating-system configuration.
- Configured persistent `KUBECONFIG`.
- Installed Helm and k9s for administrator workflows.
- Verified cluster health with `kubectl`.
- Installed Metrics Server and validated `kubectl top nodes` and `kubectl top pods`.
- Installed cert-manager using Helm.
- Verified the Helm release, CRDs, and running cert-manager pods.
- Configured and tested K3s embedded-etcd snapshots to the `genesis-k3s-etcd` RustFS bucket.
- Installed Velero v1.18.2 with the compatible AWS/S3 plugin v1.14.2 for the local RustFS endpoint.
- Validated the `genesis-velero` BackupStorageLocation as `Available`.
- Restored a deliberately deleted NGINX namespace and workload.
- Created and tested the `genesis-daily` whole-cluster resource schedule with a 14-day TTL.
- Verified 428 of 428 objects and the resulting backup artifacts in RustFS.
- Evaluated Kasten K10 9.0.2 and intentionally did not deploy it after prerequisite findings.

## Executive Summary

Designed and validated layered Kubernetes disaster recovery using K3s embedded-etcd snapshots and Velero, with local S3-compatible off-node protection, scheduled retention, and destructive restore testing. This is a development and training platform designed with enterprise architecture principles; it is not represented as a production deployment.

## High-Availability Boundary

Keepalived provides the Kubernetes API VIP at `192.168.70.5`, while HAProxy distributes API connections across three control-plane nodes. Failover of that client endpoint was deliberately tested and validated.

This does not imply that every internal control-plane relationship uses the VIP. CP-02 and CP-03 were originally joined through CP-01 at `192.168.70.10:6443`. Changing those relationships is a separate architecture review item.

## Layered Recovery Architecture

### Layer 1: K3s / embedded-etcd protection

K3s embedded-etcd snapshots protect control-plane state and support cluster disaster recovery. Manual and scheduled snapshots were validated across the control-plane environment and written directly to the local S3-compatible RustFS bucket `genesis-k3s-etcd`.

A temporary five-minute schedule was used only to observe automated S3 snapshot creation and was returned to a sensible operational schedule after testing.

The K3s server token and related configuration are required for etcd disaster recovery. Independent protection and recovery validation of that material remain explicit DR and security follow-up work; they are not claimed as completed controls.

### Layer 2: Velero resource and workload protection

Velero v1.18.2 protects Kubernetes API resources and application definitions. The compatible AWS/S3 plugin v1.14.2 supplies S3 protocol integration for the local RustFS endpoint; AWS itself is not used. Velero writes to the separate `genesis-velero` bucket, and its BackupStorageLocation was validated as `Available`.

### Layer 3: Persistent application data

The cluster uses the K3s local-path storage provisioner, but the current environment does not provide the CSI snapshot capability expected for advanced persistent-volume protection. Full persistent application-data protection has not been implemented. This is an architecture finding that informs the next storage-design phase.

## Destructive Recovery Validation

A `velero-test` namespace containing an NGINX deployment was backed up successfully and then deliberately deleted. Kubernetes confirmed that the namespace and its resources no longer existed before the restore.

Velero recreated the namespace, deployment, ReplicaSet, and a running NGINX pod. This validated the complete path:

`Kubernetes → Velero → S3-compatible RustFS → recovery`

## Automated Velero Policy

The `genesis-daily` schedule defines a whole-cluster Kubernetes resource backup, daily execution, a 14-day / `336h` TTL, and off-node storage in `genesis-velero`.

The real schedule template was tested using `genesis-daily-test` rather than waiting for its next scheduled run. The backup completed with all namespaces included, no exclusions, and 428 of 428 expected objects successfully backed up. The archive, logs, metadata, resource lists, results, volume information, and `velero-backup.json` were independently verified in `genesis-velero / backups / genesis-daily-test` through the RustFS GUI.

## Kasten K10 Evaluation

Kasten K10 9.0.2 was evaluated as a GUI and policy-driven Kubernetes data-protection option. Its official Helm repository was prepared and the official Primer was run before deployment.

The Primer found Kubernetes `v1.36.3+k3s1` outside the versions supported by that K10 release and identified the lack of CSI snapshot capability in the current local-path storage design. K10 was intentionally not deployed. This was an engineering compatibility decision, not a failed implementation.

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
HAProxy + Keepalived VMs
    - Kubernetes API VIP: 192.168.70.5
    - VIP failover validated
    |
    v
K3s Cluster: 3 control-plane nodes + workers
    |-- embedded-etcd snapshots --> RustFS / genesis-k3s-etcd
    |-- Velero resource backups --> RustFS / genesis-velero
    `-- local-path volumes -------> CSI snapshot protection not available
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
| API High Availability | Validated |
| Embedded-etcd Snapshots | Validated |
| Velero Resource Recovery | Validated |
| Ingress | Planned |
| MetalLB | Installed / Evaluating |
| GitOps | Planned |
| Observability | Planned |
| Persistent Storage | Limited: local-path, no CSI snapshots |
| Kasten K10 | Evaluated / Not Deployed |
| K3s Token and DR Configuration | Protection Validation Follow-up |
| Application Platform | Planned |

## Future Roadmap

| Phase | Focus | Components |
| --- | --- | --- |
| Phase 1 | Platform Foundation | Completed |
| Phase 2 | Control-plane HA | Completed: HAProxy, Keepalived, API VIP failover validation |
| Phase 3 | Layered Recovery | Completed: etcd snapshots, Velero, RustFS, destructive restore |
| Phase 4 | Storage and DR Hardening | CSI-capable storage, persistent-data protection, recovery-material validation |
| Phase 5 | Platform Services | Ingress, observability, GitOps, and workloads |

## Matt's Notes

What surprised me most was how quickly Kubernetes stopped being about commands and started being about boundaries. Terraform, cloud-init, bootstrap scripts, Helm, and Kubernetes each have a job. The platform gets easier to understand when each layer is allowed to do its job.

The biggest lesson was that Kubernetes is not just a thing you install. It is an operational surface. Once Metrics Server and cert-manager were installed, I had to think in terms of releases, pods, namespaces, CRDs, health checks, and upgrade paths.

This also changed how I understood my own automation history. The same philosophy I used years ago with Windows batch deployment scripts still applies here: remove repeatable manual steps, make the next run cleaner, and leave enough documentation that the process can be trusted later.

Repository organization mattered earlier than expected. At first, folders feel like ceremony. Then the project gains cloud-init files, scripts, guides, troubleshooting notes, and future roadmap items. Structure becomes part of the platform.

This project marks a real transition for me: from learning Kubernetes commands to building a reusable platform foundation that can grow into networking, observability, GitOps, storage, and eventually AI workloads.
