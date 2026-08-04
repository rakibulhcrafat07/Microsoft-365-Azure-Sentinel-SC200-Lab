# Zero-to-SOC: A Self-Funded, Multi-Certification Cloud Security Lab

**Building a real, hands-on Microsoft 365 + Azure security and administration environment — end to end, on free-tier and trial subscriptions only — in preparation for Microsoft 365 Administration, SC-200 (Security Operations Analyst), and AZ-104 (Azure Administrator).**

[![Cost](https://img.shields.io/badge/cost-%240-success)](https://img.shields.io/badge/cost-%240-success) [![Status](https://img.shields.io/badge/status-in%20progress-blue)](https://img.shields.io/badge/status-in%20progress-blue) [![Projects](https://img.shields.io/badge/projects-10%20of%2016%20complete-orange)](https://img.shields.io/badge/projects-10%20of%2016%20complete-orange)

---

## Table of contents

- [Why this exists](#why-this-exists)
- [The stack](#the-stack)
- [Project 1 — Environment Setup & Cross-Tenant Recovery](#project-1--environment-setup--cross-tenant-recovery)
- [Project 2 — Tenant, Users & Governance Foundation](#project-2--tenant-users--governance-foundation)
- [Project 3 — Identity Security: MFA, Conditional Access & Identity Protection](#project-3--identity-security-mfa-conditional-access--identity-protection)
- [Project 4 — RBAC & Delegated Administration (M365 + Azure)](#project-4--rbac--delegated-administration-m365--azure)
- [Project 5 — Azure Storage](#project-5--azure-storage-az-104)
- [Project 6 — Azure Networking: VNet, NSG, Peering & Bastion](#project-6--azure-networking-vnet-nsg-peering--bastion-az-104)
- [Project 7 — Azure Compute & On-Prem-Style AD DS for Defender for Identity](#project-7--azure-compute--on-prem-style-ad-ds-for-defender-for-identity-az-104--sc-200)
- [Project 8 — Mail Room: Exchange & Defender for Office 365](#project-8--mail-room-exchange--defender-for-office-365-m365--sc-200)
- [Project 9 — Device & Endpoint (Intune + Microsoft Defender for Endpoint)](#project-9--device--endpoint-intune--microsoft-defender-for-endpoint-sc-200--az-104)
- [Project 10 — Collaboration & Cloud App Security](#project-10--collaboration--cloud-app-security-m365--sc-200)
- [Key learnings across all ten projects](#key-learnings-across-all-ten-projects)
- [Roadmap — upcoming projects](#roadmap--upcoming-projects)
- [About](#about)

---

## Why this exists

Certification guides teach concepts in a vacuum. They don't make you sit through a declined card, discover your SIEM is quietly pointed at the wrong tenant, or figure out why a group you just built has zero members. This repository documents, screenshot by screenshot, the actual process of building a functioning multi-cloud security lab from a completely empty account — mistakes, dead ends, and fixes included — rather than a cleaned-up "final state" walkthrough.

Every stage is called a **Project** rather than a "phase" or "module," because that's what it is: a self-contained unit of work with its own goal, its own problems, and its own resolution — the same way real infrastructure work is scoped.

---

## The stack

| Component                                         | Role                                                                               |
| ------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Microsoft 365 Business Premium (trial)            | Core tenant — mail, collaboration, base licensing                                  |
| Microsoft Entra ID P2 (trial)                     | Identity governance — Conditional Access, Identity Protection, dynamic groups, PIM |
| Enterprise Mobility + Security E5 (trial)         | Defender for Identity, Defender for Cloud Apps                                     |
| Microsoft Defender for Office 365, Plan 2 (trial) | Email threat protection                                                            |
| Microsoft Defender for Endpoint, Plan 2 (trial)   | Device & endpoint threat protection                                                |
| Microsoft Intune (trial)                          | Device compliance & configuration management                                       |
| Microsoft Sentinel (Azure)                        | SIEM/SOAR — detection engineering, KQL, incident response                          |
| Azure infrastructure ($200 free credit)           | Resource Groups, RBAC, Storage, Networking, Compute                                |

---

## Project 1 — Environment Setup & Cross-Tenant Recovery

**Goal:** provision every license and service needed for the lab, on trial/free tiers only, with zero ongoing cost.

**📄 Full detail (16 screenshots, every sub-step documented): [`project-01-environment-setup/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-01-environment-setup/README.md)**

This project turned out to be less "click next a few times" and more "diagnose three separate problems in sequence":

- **Provisioned** Azure ($200 credit) and an M365 tenant, then every trial license needed (Business Premium, Entra ID P2, EMS E5, Defender for Office 365 P2)
- **Payment troubleshooting** — a debit card that verified Azure fine was repeatedly declined on Business Premium's recurring-billing model; a dual-currency credit card resolved it on the first attempt
- **Dead end** — the Microsoft 365 Developer Program rejected the account for a card-free sandbox
- **The real problem** — discovered the first Sentinel workspace had been built in a completely different tenant than the one holding every license, since Azure and M365 sign-ins don't make the active tenant obvious
- **Recovery** — rebuilt Sentinel from scratch in the correct tenant and verified all 7 Defender XDR connectors reporting Connected

[![All 7 Defender XDR connectors verified in the correct tenant](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-01-environment-setup/images/15_final_connectors_connected.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-01-environment-setup/images/15_final_connectors_connected.png)
*The moment that confirmed the rebuild worked — same tenant, fully integrated.*

---

## Project 2 — Tenant, Users & Governance Foundation

**Goal:** populate the now-correctly-scoped tenant with test identities and governance structures, and stand up the equivalent Azure administrative foundation.

**📄 Full detail (26 screenshots, every sub-step documented): [`project-02-governance-foundation/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-02-governance-foundation/README.md)**

Summary of what was built:

- **User provisioning** — 5 test identities with deliberately varied license state, department, and job title (used as test data throughout later projects)
- **Security Group** (assigned membership) — `IT-Support-Team`
- **Microsoft 365 Group** — `Managing-Team`, verified it auto-provisioned a linked Team and SharePoint site
- **Dynamic Group** — a real debugging story: the M365 admin center's group wizard only supports Assigned membership; dynamic rules had to be configured separately in the Entra admin center (`(user.department -eq "IT")`), and membership evaluation turned out to be asynchronous rather than instant
- **Azure governance foundation** — explored the Management Group → Subscription hierarchy, then created a tagged Resource Group (`environment: lab`, `owner: rakibul`) as the scoping unit for every later Azure project

[![Dynamic membership rule configured in Entra admin center](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-02-governance-foundation/images/19-dynamic-rule-syntax.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-02-governance-foundation/images/19-dynamic-rule-syntax.png)
*The one screenshot that matters most from this project — the M365 admin center simply doesn't expose this screen.*

---

## Project 3 — Identity Security: MFA, Conditional Access & Identity Protection

**Goal:** move beyond password-only sign-in by layering per-user MFA, group-scoped Conditional Access, location-based access, and risk-based policies on top of the Project 1–2 tenant — then verify with logs that the policies actually fire.

**📄 Full detail (31 screenshots, every sub-step documented): [`project-03-identity-security/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-03-identity-security/README.md)**

Summary of what was built:

- **Per-user MFA** enabled for test user `Shakhawat Hossian Anik`, as a baseline before moving to Conditional Access
- **CA01** — required MFA for the existing `IT-Support-Team` group (reused rather than duplicated) across all cloud apps
- **CA02** — location-based access: a `Bangladesh-Trusted` named location, MFA required everywhere except it
- **CA03 / CA04** — sign-in risk and user risk policies, rebuilt as Conditional Access after discovering the classic Identity Protection risk policy pages are being retired October 1, 2026
- **Verification** — 5 deliberate failed logins, confirmed end-to-end via Sign-in logs (error `50126`) and the Conditional Access Policy Impact tab, all policies kept in **Report-only** throughout

[![All four Conditional Access policies, Report-only](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-03-identity-security/images/06_all_4_ca_policies.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-03-identity-security/images/06_all_4_ca_policies.png)
*CA01 through CA04, all evaluated in Report-only mode before anything gets enforced.*

---

## Project 4 — RBAC & Delegated Administration (M365 + Azure)

**Goal:** move from "everyone is Global Admin / Owner" to scoped, least-privilege access — per-role M365 admin delegation, PIM just-in-time activation, Azure RBAC (built-in and custom), and a hard guardrail (Resource Lock) that survives even Owner-level access.

**📄 Full detail (22 screenshots, every sub-step documented): [`project-04-rbac-iam/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-04-rbac-iam/README.md)**

Summary of what was built:

- **M365 User Administrator role** — assigned and verified: could reset passwords and create users, blocked from granting Global Admin
- **Administrative Unit** — a scoped admin whose Active Users list only showed in-scope members; accounts outside the AU didn't just become un-editable, they weren't in the list at all
- **PIM eligible role** — Exchange Administrator set to Eligible (not Active); confirmed the account was blocked before activation, then got full Exchange admin center access the moment it self-activated
- **Azure RBAC — Reader** — could view a resource group, blocked from creating any resource in it
- **Custom RBAC role** ("VM Operator") — built from just 3 Actions (start/restart/stop); the assigned user hit a 401 even opening the resource group blade, since narrowly-scoped custom roles don't inherit the generic `resourceGroups/read` permission that Reader carries as a wildcard
- **Resource Lock** — a Delete lock that blocked deletion even when attempted from the tenant's own Global Administrator / Owner account

[![Delete blocked by lock, even for the account's own Owner role](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-04-rbac-iam/images/s24_delete_blocked_by_lock.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-04-rbac-iam/images/s24_delete_blocked_by_lock.png)
*Locks sit above RBAC entirely — no role assignment, however privileged, overrides one.*

---

## Project 5 — Azure Storage (AZ-104)

**Goal:** move from access-control theory into a mechanic AZ-104 tests directly — Blob storage and tiering, a real mountable Azure File Share, scoped time-boxed access via SAS tokens, an automated cost-lifecycle policy, and soft delete recovery — each one tested end-to-end rather than just configured and screenshotted.

**📄 Full detail (38 screenshots, every sub-step documented): [`project-05-storage-management/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-05-storage-management/README.md)**

Summary of what was built:

- **Storage account + Blob container** — `strakibullab01`, container `mycontainer`, a test file uploaded and downloaded; caught the account defaulting to Geo-redundant storage (GRS) and switched to Locally-redundant (LRS) before creation
- **Access tier change** — moved a blob from Hot to Cool, verified via the ACCESS TIER LAST MODIFIED timestamp that the change actually committed rather than just appearing to
- **Azure File Share** — `myfileshare` mounted as a real Windows network drive; the creation wizard tried to silently enable Azure Backup by default (unchecked before creating), and the mount itself succeeded in an Administrator PowerShell session but didn't appear in File Explorer until re-run in a normal session — a genuine admin-vs-user drive-mapping gap
- **SAS token** — generated a read-only, HTTPS-only, time-boxed Shared Access Signature and tested the URL live in a private browser tab with zero Azure login
- **Lifecycle management policy** — a single rule automating the full cost lifecycle: 30 days → Cool, 90 days → Archive, 365 days → Delete
- **Soft delete** — deleted a blob on purpose, confirmed it disappeared from the container's default "active blobs" view, then recovered it via Undelete with identical hash, tier, and size intact

[![SAS URL opened directly in a private browser tab, no login required](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-05-storage-management/images/24_sas-url-tested-in-browser.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-05-storage-management/images/24_sas-url-tested-in-browser.png)
*Read-only, time-boxed, zero credentials exchanged — the SAS token doing exactly what it's supposed to.*

---

## Project 6 — Azure Networking: VNet, NSG, Peering & Bastion (AZ-104)

**Goal:** build the core network layer for the lab — a segmented VNet, a security-group-filtered subnet, a public IP, peered connectivity between two VNets, and Azure Bastion for remote access that never requires a VM to carry its own public IP. AWS VPC was dropped from this phase entirely; the lab stays Azure/Microsoft-only going forward.

**📄 Full detail (90 screenshots, every sub-step documented): [`project-06-networking-full/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-06-networking-full/README.md)**

Summary of what was built:

- **VNet + 2 subnets** — `vnet-lab-sc200` (`10.10.0.0/16`) split into `subnet-app` (`10.10.1.0/24`) and `subnet-data` (`10.10.2.0/24`)
- **NSG with inbound/outbound rules** — `nsg-app-subnet`, with SSH (22) restricted to a single source IP rather than the open internet, an explicit outbound HTTPS (443) rule, and the NSG associated directly to `subnet-app`
- **Public IP** — a standalone Standard-SKU static IP, provisioned for later use
- **VNet Peering** — a second VNet (`vnet-lab-sc200-peer`, non-overlapping `10.20.0.0/16`) peered with the first, verified **from both sides** (each VNet shows its own peering link name and "Connected" status independently)
- **Azure Bastion** — reviewed the configuration first, then actually deployed it, spun up a no-public-IP test VM, and connected to it live through the browser — confirmed with a real shell prompt, not just a green checkmark — before deleting everything to stop the hourly billing

Along the way: a home ISP that rotated its public IP mid-session and broke the just-configured NSG rule, three VM sizes rejected as unavailable before one finally deployed, and a resource-group Delete lock (left over from Project 4) that silently blocked the cleanup until it was found, lifted, and restored afterward.

[![Connected via Bastion — a live shell, no public IP on the VM](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-06-networking-full/images/078.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-06-networking-full/images/078.png)
*azureuser@vm-test-linux:~$ — proof the connection works without the VM ever being exposed to the internet.*

---

## Project 7 — Azure Compute & On-Prem-Style AD DS for Defender for Identity (AZ-104 / SC-200)

**Goal:** stand up the core compute layer on top of the Project 6 network — a Windows Server VM and a Linux VM, resized and snapshotted like real production workloads — then deliberately promote the Windows VM to an Active Directory Domain Controller so it generates the kind of real, on-prem-style identity telemetry that Microsoft Defender for Identity is actually built to monitor, rather than relying on synthetic data.

**📄 Full detail (all steps documented): [`project-07-azure-compute/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-07-azure-compute/README.md)**

Summary of what was built:

- **Two VMs deployed into `vnet-lab-sc200`/`subnet-app`** from Project 6 — a Windows Server VM (`vm-win-addc`) and an Ubuntu Linux VM (`vm-ubuntu-lab`)
- **VM resize, managed disk & snapshot** — resized a running VM to a different size, then created a managed disk snapshot as a point-in-time backup mechanic
- **Custom Script Extension** — deployed automation directly onto a VM at runtime, reusing the Project 5 storage account (`strakibullab01`/`mycontainer`) as the script's source
- **Active Directory promotion** — `vm-win-addc` promoted to a Domain Controller for a brand-new forest, `rakibul.local`, using the standard AD DS Configuration Wizard
- **Microsoft Defender for Identity sensor** — installed and verified on the new domain controller, registered correctly as a domain-controller-type sensor for `rakibul.local`
- **Cost control** — both VMs stopped (deallocated) once verification was complete, to avoid ongoing compute charges between sessions

Along the way: a first-time Defender for Identity workspace provisioning failure that turned out to be a temporary Microsoft-side backend delay rather than a licensing problem, an initial attempt at the newer "Activate servers" sensor method that was abandoned in favor of the classic sensor installer (since "Activate servers" requires separate Defender for Endpoint device onboarding), and a SAS-token-based workaround to get the sensor installer file onto the VM through Azure Bastion, which has no native file-upload feature of its own.

*Highlight: a live Defender for Identity sensor reporting against a real Active Directory forest — `rakibul.local` — built specifically to feed realistic identity signals into the SC-200 detection projects still ahead on the roadmap.*

---

## Project 8 — Mail Room: Exchange & Defender for Office 365 (M365 / SC-200)

**Goal:** build and secure the mail layer of the tenant — a shared mailbox, a mail flow rule, and message trace verification in Exchange admin center, followed by custom anti-phishing, anti-spam, and anti-malware policies, a real EICAR anti-malware detection test, a Safe Links URL-scanning test, and a full end-to-end phishing attack simulation in Microsoft Defender for Office 365.

**📄 Full detail (99 screenshots, every sub-step documented): [`project-08-mail-protection/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-08-mail-protection/README.md)**

Summary of what was built:

- **Shared mailbox** — `IT Support`, with Md. Imran Ahmed and Pallob Kumar Roy granted Full Access as delegates
- **Mail flow rule** — a subject-keyword rule that Bcc's the admin whenever "confidential" appears, deployed in Audit (Test) mode first, matching this lab's established Report-only discipline
- **Message trace** — verified end-to-end delivery of a test message and confirmed (with a short reporting delay) that the mail flow rule had genuinely executed
- **Custom threat policies** — a dedicated anti-phishing policy with per-user impersonation protection, a stricter anti-spam policy with a lowered bulk-email threshold, and an anti-malware policy with the common attachments filter and zero-hour auto purge enabled
- **EICAR test** — sent the industry-standard, harmless EICAR test file as an attachment and confirmed it was automatically quarantined with reason: **Malware**
- **Safe Links test** — created a custom Safe Links policy with real-time URL scanning enabled across Email, Teams, and Office apps
- **Attack Simulation Training** — ran a full Credential Harvest phishing simulation against all four test accounts, observed the phishing email land, clicked through to the fake credential page, was redirected to the education landing page, and confirmed the outcome in the admin report: 1 click, 1 compromised credential

Along the way: two real backend errors were hit and resolved — an anti-phishing policy that failed to save until an explicit quarantine policy was selected for the "Quarantine the message" action, and a browser that opened the EICAR test file as plain text instead of downloading it, fixed by using the official zipped package instead.

[![Quarantine list confirms the anti-malware policy caught the EICAR test file](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-08-mail-protection/images/86.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-08-mail-protection/images/86.png)
*Quarantine reason: Malware — direct proof that the custom anti-malware policy built earlier in this project is actually working.*

---

## Project 9 — Device & Endpoint (Intune + Microsoft Defender for Endpoint) (SC-200 / AZ-104)

**Goal:** stand up an Intune baseline for Windows devices (compliance + configuration), onboard the Project 7 domain controller VM into Microsoft Defender for Endpoint, and explore the Exposure Management / Vulnerability Management dashboards.

**📄 Full detail (48 screenshots, every sub-step documented): [`project-09-device-endpoint/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-09-device-endpoint/README.md)**

Summary of what was built:

- **Compliance Policy** — `CP-Windows10-Baseline`, defining the minimum security bar a device must meet
- **Configuration Profile** — `CFG-Windows10-SecurityBaseline` via Settings Catalog, enforcing BitLocker, Firewall across all three network profiles, Antivirus, and Real-time protection
- **Defender for Endpoint onboarding** — the Project 7 domain controller VM (`vm-win-addc`, Windows Server 2022) onboarded via local onboarding script, reached through a re-created Azure Bastion session
- **Exposure Management & Vulnerability Management** — explored both dashboards to see how device risk and known vulnerabilities are surfaced once a device is onboarded
- **EICAR test intentionally skipped** — already demonstrated at the mail layer in Project 8, so it wasn't repeated here

Along the way: the profile-type dropdown initially accepted "Templates" — a category header, not an actual policy type — as a valid selection; the Real-time protection toggle stayed greyed out until its parent Microsoft Defender Antimalware setting was turned on first; a Settings Catalog checkbox added a setting to the profile without actually setting its value; the Defender onboarding package was first generated for the wrong OS family (Windows 10/11 instead of Windows Server 2019/2022/2025); and the freshly onboarded VM briefly appeared as two separate entries in Device Inventory.

[![vm-win-addc successfully onboarded into Microsoft Defender for Endpoint](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-09-device-endpoint/images/48.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-09-device-endpoint/images/48.png)
*The domain controller built back in Project 7 is now a monitored endpoint, not just an identity source.*

---

## Project 10 — Collaboration & Cloud App Security (M365 / SC-200)

**Goal:** stand up a standalone SharePoint site and test its external-sharing policy, schedule a Teams meeting with an external guest to compare against SharePoint's sharing behavior, explore Microsoft Defender for Cloud Apps end-to-end (Activity log, App Connectors, Governance), and build a custom File Policy that flags externally-shared files.

**📄 Full detail (62 screenshots, every sub-step documented): [`project-10-collaboration-security/README.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-10-collaboration-security/README.md)**

The concept behind this project: once a file or a meeting leaves an organization's boundary, three separate layers have to work together — the **Policy** set in advance, the **Enforcement** the moment someone actually shares, and the **Visibility** tooling that watches for it afterward. Testing all three back to back on the same data isolated a real fault:

- **SharePoint site + external sharing** — a standalone Communication site (`ExternalShare-Test`) with site-level sharing set to *New and existing guests*; a share invite to an external Gmail address showed a "success" confirmation that turned out to be false — *Manage Access* later showed the file had never actually been shared with anyone
- **Teams meeting + external access** — the identical external Gmail address was invited to a Teams meeting instead, and it worked perfectly, arriving with full RSVP controls — proving the failure was specific to SharePoint's sharing pipeline, not a tenant-wide guest access problem
- **Defender for Cloud Apps** — discovered the Office 365 App Connector had never been provisioned (hence an empty Activity log); connecting it immediately produced an HTTP 400 error traced through three different Microsoft Purview surfaces (one retired since November 2024) to a broken Unified Audit Log pipeline
- **File Policy** — a custom policy (`Flag-Externally-Shared-Files`) built to auto-flag files shared externally or publicly, using a non-destructive "notify file owner" action consistent with this lab's Report-only-first discipline

[![Flag-Externally-Shared-Files policy created and Active](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/62.png)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-10-collaboration-security/images/62.png)
*A working File Policy, built despite the audit pipeline issues uncovered earlier in the same project.*

---

## Key learnings across all ten projects

1. **Payment failures can be bank-side, not account-side.** A domestic debit card that verifies a one-time Azure hold can still be blocked on a recurring-billing merchant. A dual-currency credit card resolved it reliably.
2. **Always confirm which tenant is active before provisioning.** Azure and Microsoft 365 sessions can silently diverge into different directories even when the sign-in usernames look identical — this cost an entire rebuild.
3. **The simplified admin UI is not the full picture.** The Microsoft 365 admin center covers common tasks well, but advanced identity governance (dynamic groups, PIM, administrative units) lives in the Entra admin center.
4. **Background processes aren't instant.** Dynamic group membership, license propagation, and connector activation can all take several minutes — don't assume a misconfiguration too early.
5. **Tag at creation, not after.** Applying `environment`/`owner` tags when a Resource Group is created (rather than retrofitting later) keeps cost tracking clean once Storage and Compute projects begin.
6. **Report-only is the safe default for new Conditional Access policies.** It confirms a policy is evaluated correctly against real sign-in activity before it can lock anyone out.
7. **Microsoft's own tooling is mid-migration.** Classic Identity Protection risk policies are being sunset in favor of Conditional Access — worth knowing since the live portal won't always match older certification material.
8. **Scoping restricts visibility, not just actions.** Both the Administrative Unit and the custom RBAC role showed accounts/resources disappearing from view entirely, rather than throwing an access-denied error.
9. **Eligible ≠ Active, and it's easy to get one when you meant the other.** This tenant's Entra ID P2 license extends PIM governance to Azure resource roles by default — two separate role assignments in Project 4 landed as Eligible when Active was intended.
10. **A Resource Lock is the strongest guardrail in the whole RBAC toolkit** — strong enough that it later blocked its owner's own cleanup work in Project 6 until it was deliberately lifted.
11. **Portal wizards default to the option that costs more or does more than asked.** Geo-redundant storage over Locally-redundant, and an auto-enabled backup vault during file share creation, both surfaced in Project 5 alone — worth a second look on every Create screen, not just these two.
12. **A soft-deleted resource often isn't visible in the default UI view**, which can look identical to a permanent delete if the filter itself isn't known. Visibility settings deserve as much attention as the safety feature underneath them.
13. **A subnet-level NSG rule protects any resource later placed in that subnet** — no separate NSG was needed at the VM level once `subnet-app` already had one attached.
14. **Inbound and outbound NSG rules are not symmetric by default.** Azure's built-in rules end in an implicit Deny All for inbound traffic, but already allow outbound internet access — inbound needs explicit rules far more urgently than outbound does.
15. **VM size availability is subscription- and region-specific, not universal.** Three different low-cost sizes were rejected as unavailable in Project 6 before one finally deployed — worth checking availability before planning around a specific SKU.
16. **Deleting a resource doesn't always delete what it created.** Azure Bastion's auto-generated public IP survives Bastion's own deletion and has to be cleaned up as a separate step.
17. **Building real on-prem-style infrastructure inside Azure pays off later.** Promoting a VM to an actual Active Directory Domain Controller (rather than skipping straight to cloud-only identity) is what makes Microsoft Defender for Identity's sensor produce genuine telemetry for the detection projects still ahead.
18. **A "no data" backend error can be a temporary delay, not a real failure.** The first Defender for Identity workspace provisioning attempt failed, then succeeded on retry with no configuration changes — worth ruling out a transient Microsoft-side issue before troubleshooting further.
19. **Not every "newer" method is the right one for the current setup.** Defender for Identity's newer "Activate servers" sensor method assumes Defender for Endpoint device onboarding is already in place; the classic sensor installer was the correct choice here instead.
20. **Exchange mail flow rules are disabled by default after creation** and must be manually enabled before they will actually process anything.
21. **Anti-phishing, anti-spam, and anti-malware are three genuinely separate detection layers**, each with its own action pipeline (reject, quarantine, move to junk) — full mail protection needs all three configured, not just the tenant defaults.
22. **The EICAR test string is completely harmless and industry-standard**, but it still has to be delivered as a real file — some browsers will render a `.com`/`.txt` test file as plain text instead of downloading it, which looks like a failure but isn't one.
23. **Wizard-level validation errors don't always explain themselves.** A generic "Client Error" on the anti-phishing policy wizard turned out to trace back to one specific unset dropdown (the quarantine policy) several steps back in the flow.
24. **Attack Simulation Training's full lifecycle — delivery, click, credential harvest, education, and reporting — can be observed end to end** using only the lab's existing test accounts, at zero additional cost.
25. **A dropdown accepting a wrong-but-plausible value is a real failure mode.** Intune's profile-type picker let a category header ("Templates") through as if it were an actual policy type — worth double-checking the selection, not just that *something* got selected.
26. **Some UI toggles have invisible dependencies.** The Real-time protection setting in Intune stayed greyed out until its parent Microsoft Defender Antimalware setting was enabled first — no inline explanation pointed to this.
27. **Adding a setting and configuring its value are two different actions.** A Settings Catalog checkbox can add a setting to a profile without actually turning its value on — the profile can look complete while doing nothing.
28. **Onboarding packages are OS-family specific, not universal.** Generating a Defender for Endpoint onboarding script for the wrong OS family (Windows 10/11 instead of Windows Server) produces a package that won't onboard the intended machine.
29. **A freshly onboarded device can briefly appear as duplicate entries in inventory** — worth waiting a sync cycle before assuming an onboarding failure.
30. **A "success" confirmation in the UI is not proof of a persisted state.** SharePoint's share dialog showed a clean success message for an invite that never actually took effect — always verify through a second, independent view before trusting the first confirmation.
31. **Alert-level integration and activity-level ingestion are different systems.** Defender XDR connectors verified as "Connected" say nothing about whether Defender for Cloud Apps' Activity log or App Connectors are separately provisioned and populated.
32. **Site-level SharePoint sharing policy constrains what's even offered at the file level**, enforcing the ceiling automatically downstream at the individual file-share dialog.
33. **Comparing two features against the same test input isolates the fault.** Sending an invite to the same external email through SharePoint (failed) and Teams (succeeded) in the same session proved the problem was SharePoint-specific rather than tenant-wide.
34. **Microsoft's compliance/audit surface is mid-migration.** A single troubleshooting chain hit a new Purview governance portal error, a classic compliance portal retired since November 2024, and a current Audit search page that itself failed to load.
35. **Some governance actions have silent prerequisites.** File Policy options like "Put in admin quarantine" and "Apply sensitivity label" stay greyed out until their supporting infrastructure (a quarantine location, a defined label taxonomy) exists.
36. **Report-only discipline extends naturally to Cloud App governance actions** — choosing a non-destructive "notify" action first is consistent with the Audit/Report-only-first pattern used for Conditional Access and mail flow rules earlier in this lab.

---

## Roadmap — upcoming projects

| #  | Project                                                          | Certification focus |
| --- | ---------------------------------------------------------------- | ------------------- |
| 1  | Environment Setup & Cross-Tenant Recovery                        | ✅ Complete          |
| 2  | Tenant, Users & Governance Foundation                            | ✅ Complete          |
| 3  | Identity Security — MFA, Conditional Access, Identity Protection | ✅ Complete          |
| 4  | RBAC & Delegated Administration (M365 + Azure)                   | ✅ Complete          |
| 5  | Azure Storage                                                    | ✅ Complete          |
| 6  | Azure Networking — VNet, NSG, Peering, Bastion                   | ✅ Complete          |
| 7  | Azure Compute (+ on-prem-style AD DS for Defender for Identity)  | ✅ Complete          |
| 8  | Email Security — Exchange + Defender for Office 365              | ✅ Complete          |
| 9  | Endpoint Security — Intune + Defender for Endpoint               | ✅ Complete          |
| 10 | Collaboration & Cloud App Security                               | ✅ Complete          |
| 11 | Data Protection & Compliance                                     | M365 (Next)          |
| 12 | Monitoring — Azure Monitor + Sentinel                            | AZ-104 / SC-200     |
| 13 | KQL (Kusto Query Language)                                       | SC-200              |
| 14 | Detection Engineering — Analytics Rules                          | SC-200              |
| 15 | Incident Investigation                                           | SC-200              |
| 16 | Automation, Backup & Cost Management                             | AZ-104 / SC-200     |

Full task-level detail for every upcoming project: [`ROADMAP.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/ROADMAP.md)

---

## About

Maintained by **Rakibul Hoque Chowdhury**, pursuing CEH → OSCP → OSDA alongside this project.
