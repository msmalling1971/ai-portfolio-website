# Microsoft Endpoint Management Proof of Concept

## Executive Summary

This privately maintained proof of concept evaluated Microsoft cloud endpoint-management controls on a Windows 11 Enterprise endpoint. The work connected Microsoft Entra join, Microsoft Intune enrollment, pilot-group assignment, BitLocker, Microsoft Defender Firewall, Microsoft Defender Antivirus, PowerShell validation, and policy rollback.

This was one component of a broader Azure Enterprise learning and evaluation initiative designed to recreate technologies and operational scenarios commonly managed by Infrastructure and Platform Engineering teams. The objective was not to deploy isolated features, but to understand how identity, endpoint management, security baselines, authentication, and troubleshooting interact as one system.

Policies were assigned first to `GRP-Test-Devices`. The controlled scope exposed two useful failure modes: a firewall baseline interrupted RDP until explicit service rules were deployed, and a user-targeted Intune policy blocked Windows Settings. Both were diagnosed from the endpoint and corrected through centrally managed policy. This was an enterprise-style lab evaluation, not a production deployment or corporate fleet rollout.

## Why This Project Matters

As a former Head of Infrastructure Engineering, I built this proof of concept as part of my transition into modern Microsoft cloud endpoint management. Rather than relying only on documentation or certification material, I wanted to validate how Microsoft Entra ID, Intune, endpoint security, identity, and policy enforcement behave together in a controlled enterprise-style environment.

## Business Context

Endpoint policy is useful only when its effective behavior matches the intended design. A successful assignment in an administrative portal does not independently prove that encryption, firewall rules, antivirus controls, authentication, and user restrictions are operating correctly together.

The environment used Microsoft 365 E3 licensing and evaluated Microsoft Entra ID, Microsoft Intune, BitLocker, Microsoft Defender Firewall, and Microsoft Defender Antivirus capabilities available within the tenant. Licensing and service availability were verified in the tenant at the time of the proof of concept. The presence of a service plan was not treated as evidence that every related capability had been deployed or exercised.

## Scope and Constraints

The evaluation covered one Windows 11 Enterprise evaluation endpoint, Microsoft Entra join, Intune enrollment, device-policy assignment through `GRP-Test-Devices`, BitLocker, Defender Firewall, Defender Antivirus, endpoint-side validation, and rollback of a user-targeted policy. The endpoint was hosted as a Windows 11 Enterprise virtual machine on Proxmox to provide a repeatable enterprise-style testing environment.

Constraints were explicit:

- The environment was privately maintained and was not connected to a corporate fleet.
- Validation covered one controlled endpoint rather than multiple hardware models or deployment rings.
- Policies and services reflected tenant availability at the time of testing.
- No production impact, cost reduction, migration outcome, or business return was measured.
- Defender for Endpoint onboarding is not claimed.
- Autopilot, Conditional Access, Windows LAPS, Attack Surface Reduction rules, and update rings were not implemented as part of this work.

## Architecture

```text
Microsoft 365 Tenant
|-- Microsoft Entra ID
|   |-- Device identity
|   |-- BitLocker recovery-key escrow
|   `-- Web-account RDP authentication
`-- Microsoft Intune
    `-- GRP-Test-Devices
        |-- BitLocker
        |-- Defender Firewall
        |-- Firewall Rules
        `-- Defender Antivirus

Both management paths converge on:
Windows 11 Enterprise VM / Proxmox-hosted / CL-WIN11-002
    `-- PowerShell and Endpoint Validation
```

`GRP-Test-Devices` provided the initial assignment boundary. Microsoft Entra ID held the device identity and BitLocker recovery information and participated in the tested web-account RDP authentication path. Intune delivered the endpoint-security policies. Local commands validated the effective endpoint state.

## Engineering Principles Applied

- Pilot before production.
- Trust endpoint evidence over portal status.
- Separate security baselines from service exceptions.
- Correct configuration at the management source.
- Scope exceptions as narrowly as possible.
- Validate each layer independently.

## Policy Design

### Pilot Assignment

Security policies were assigned first to `GRP-Test-Devices`. The pilot limited the effect of incomplete policy design and created a known endpoint on which to correlate assignment, synchronization, and effective state.

### BitLocker

BitLocker was deployed through Intune Endpoint Security with TPM 2.0 protection, a numerical recovery-password protector, and recovery information escrow to Microsoft Entra ID. The endpoint's encryption and protector state was validated locally. Recovery passwords and recovery-key identifiers are sensitive and must not be included in public evidence.

### Microsoft Defender Firewall

The baseline enabled the Domain, Private, and Public profiles; blocked inbound traffic by default; allowed outbound traffic by default; configured logging; and restricted local policy merge where appropriate.

Required exceptions were implemented separately so their purpose and scope could be reviewed without changing the baseline.

| Policy | Direction | Protocol | Port | Scope | Purpose |
| --- | --- | --- | --- | --- | --- |
| RDP inbound rule | Inbound | TCP | 3389 | Required management path | Permit Remote Desktop transport |
| Lab name-resolution exception | Inbound | UDP | 137 | Local lab subnet | Temporarily restore the tested short-name resolution path |

During testing, the selected authentication and name-resolution workflow exposed a dependency on legacy NetBIOS name resolution within the lab. A temporary, narrowly scoped inbound UDP 137 rule was created to understand the interaction and restore functionality while the dependency was investigated.

NetBIOS is legacy. The rule was a documented lab exception and technical-debt item, not a recommended production design. Modern organizations should prefer managed DNS. The long-term improvement is to remove the UDP 137 dependency.

### Microsoft Defender Antivirus

The antivirus baseline enabled real-time monitoring, behavior monitoring, cloud-delivered protection, download and attachment scanning, script scanning, on-access protection, automatic safe sample submission, Network Protection in block mode, and Potentially Unwanted Application protection in block mode. No broad antivirus exclusions were configured.

### User-Targeted Settings Policy

The user-targeted policy `WIN11-CFG-Hide-Settings-App` configured `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\NoControlPanel`. With `NoControlPanel=1`, both Control Panel and Windows Settings were blocked for the targeted user. This user-context behavior was distinct from the device-targeted Endpoint Security policies.

## Implementation

1. Created the Windows 11 Enterprise endpoint `CL-WIN11-002`, joined it to Microsoft Entra ID, and enrolled it in Intune.
2. Added the endpoint to `GRP-Test-Devices` for bounded policy assignment.
3. Deployed BitLocker and validated encryption, TPM protection, the numerical recovery protector, and Entra recovery escrow.
4. Deployed the Defender Firewall baseline and confirmed default inbound blocking.
5. Added separately managed TCP 3389 and subnet-scoped UDP 137 rules after validating the dependencies.
6. Deployed the Defender Antivirus baseline and inspected both configuration and operational status.
7. Traced the Windows Settings restriction to its user-targeted Intune policy and corrected it at the source.

## Validation and Evidence

> Administrative status was used to understand assignment and processing. Endpoint-side checks were used to validate effective state.

The following commands show the validation methods, not fabricated output.

### Identity

```powershell
dsregcmd /status
```

Relevant fields included `AzureAdJoined`, `DeviceId`, `TenantId`, and device and user authentication state.

### BitLocker

```powershell
Get-BitLockerVolume -MountPoint "C:" |
    Select-Object MountPoint, VolumeStatus, ProtectionStatus, EncryptionPercentage, KeyProtector

manage-bde -status C:
manage-bde -protectors -get C:
```

These commands validated volume encryption, protection status, and the TPM and numerical recovery-password protectors.

### Firewall

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction,
        LogAllowed, LogBlocked, LogFileName

Get-NetFirewallRule |
    Where-Object DisplayName -Match "RDP|NetBIOS" |
    Select-Object DisplayName, Enabled, Direction, Action, Profile, PolicyStoreSourceType
```

Port and subnet scope were correlated with `Get-NetFirewallPortFilter` and `Get-NetFirewallAddressFilter`.

### Transport and Name Resolution

```powershell
Test-NetConnection -ComputerName "CL-WIN11-002" -Port 3389 -InformationLevel Detailed
Test-NetConnection -ComputerName "<endpoint-ip-address>" -Port 3389
```

The IP test isolated TCP transport from hostname resolution and identity-dependent authentication.

### Microsoft Defender Antivirus

```powershell
Get-MpPreference |
    Select-Object DisableRealtimeMonitoring, DisableBehaviorMonitoring,
        DisableIOAVProtection, DisableScriptScanning, MAPSReporting,
        SubmitSamplesConsent, EnableNetworkProtection, PUAProtection,
        ExclusionPath, ExclusionProcess, ExclusionExtension

Get-MpComputerStatus |
    Select-Object AntivirusEnabled, AntispywareEnabled, RealTimeProtectionEnabled,
        BehaviorMonitorEnabled, IoavProtectionEnabled, OnAccessProtectionEnabled,
        NISEnabled, AntivirusSignatureVersion, AntivirusSignatureLastUpdated
```

`Get-MpPreference` validated configuration; `Get-MpComputerStatus` validated operational state.

### User Policy

```powershell
Get-ItemProperty `
    -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
    -Name "NoControlPanel" `
    -ErrorAction SilentlyContinue
```

The registry was inspected as evidence. A manual registry edit was not retained as remediation.

### Recommended Screenshot Evidence

Up to six authentic, redacted screenshots may be added later:

1. Microsoft Entra device record.
2. Intune managed-device overview.
3. BitLocker recovery escrow or protector evidence.
4. Firewall baseline and rule assignment.
5. Defender Antivirus policy.
6. PowerShell validation output.

Screenshots must redact recovery passwords, recovery-key identifiers, tenant IDs, unnecessary device IDs, usernames, email addresses, and sensitive IP details. No screenshot is claimed to exist until authentic evidence is available.

## Troubleshooting and Recovery

### RDP Transport, Name Resolution, and Entra Authentication

The failure crossed transport, name resolution, authentication, and device identity:

1. The firewall baseline enforced default inbound blocking.
2. RDP failed because TCP 3389 was not yet explicitly allowed.
3. A dedicated inbound TCP 3389 rule restored the network path.
4. Connectivity by IP succeeded.
5. Microsoft Entra web-account authentication still required the registered short device name.
6. The short name no longer resolved because the firewall baseline blocked the lab's legacy NetBIOS name-resolution path.
7. A narrowly scoped inbound UDP 137 rule restored short-name resolution.
8. RDP succeeded using `CL-WIN11-002` with **Use a web account to sign in to the remote computer**.

The successful IP test proved TCP transport. It did not validate hostname resolution, Entra authentication, or device identity. UDP 137 restored the lab workflow but remained temporary technical debt to be replaced by managed DNS.

### Windows Settings Rollback

The user-targeted `WIN11-CFG-Hide-Settings-App` policy set `NoControlPanel=1`, blocking Control Panel and Windows Settings. The setting was confirmed in the affected user's HKCU context, correlated with the applied Intune policy, and corrected at the management source. After reevaluation, the registry value was removed and Settings returned without rebuilding the endpoint.

## Engineering Decisions

| Decision | Reason | Tradeoff | Outcome |
| --- | --- | --- | --- |
| Assign to `GRP-Test-Devices` first | Access and UX failures needed a bounded scope | Production-like expansion waited for validation | Issues remained limited to the pilot endpoint |
| Separate baseline and service rules | Default posture and exceptions have different lifecycles | More policy objects require ownership | Exceptions became independently reviewable |
| Block inbound traffic by default | Access should require an explicit need | Missing dependencies interrupt services | RDP and name-resolution requirements became visible |
| Scope UDP 137 to the lab subnet | Testing exposed a legacy dependency | Temporary technical debt remained | Functionality returned without broad NetBIOS exposure |
| Validate effective endpoint state | Portal status cannot show every interaction | Commands require endpoint access | Identity, encryption, firewall, antivirus, and registry state were confirmed |
| Correct `NoControlPanel` in Intune | Local changes create drift | Reevaluation may not be immediate | Settings returned with policy authority intact |
| Avoid broad antivirus exclusions | General exclusions weaken protection | Compatibility needs narrower analysis | Broad Defender coverage remained in place |

## Operational Lessons

- **Pilot before production.** Security policy can interrupt required access even when it behaves exactly as configured.
- **Keep baselines separate from exceptions.** Each allow rule should have a visible reason and scope.
- **Validate the complete path.** Transport, name resolution, authentication, and identity are related but distinct.
- **Use endpoint evidence.** Portal reporting should be correlated with effective local state.
- **Fix managed policy at the source.** Local overrides conceal errors and create drift.
- **Minimize legacy dependencies.** Temporary NetBIOS access should remain scoped and tracked for removal.
- **Distinguish user and device targeting.** Assignment and processing context determine how a setting applies.

## Technologies & Engineering Skills

### Technologies

- Microsoft Entra ID
- Microsoft Intune
- Windows 11 Enterprise
- Microsoft Defender Antivirus
- Microsoft Defender Firewall
- BitLocker
- PowerShell

### Engineering Skills

- Endpoint Security
- Identity and Device Management
- Policy Design
- Pilot Deployment Strategy
- Endpoint Validation
- Policy Troubleshooting
- Change Control
- Rollback Discipline

## Future Improvements

- Replace the lab's NetBIOS dependency with managed DNS.
- Test a second endpoint to compare physical and virtual TPM behavior.
- Evaluate Windows Autopilot provisioning.
- Integrate compliance policy with Conditional Access.
- Evaluate Windows LAPS and Attack Surface Reduction rules using an audit-first approach.
- Design formal test, pilot, update, and broader deployment rings.
- Define policy naming, versioning, ownership, export, and rollback standards.
- Add carefully redacted evidence screenshots.

These are planned evaluation areas, not completed capabilities.

## Executive Takeaway

The value of this project was not that policies deployed successfully. It was understanding how identity, endpoint security, authentication, validation, and troubleshooting behave together when things do not go as planned.
