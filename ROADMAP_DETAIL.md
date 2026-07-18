# Roadmap Detail — Project 3 through Project 16 (M365 Admin + SC-200 + AZ-104)

Rather than keeping the three certification tracks (M365 Admin, SC-200, AZ-104) separate, they're sequenced into **one realistic order** — the way a company's IT infrastructure, user management, and security actually get built together, not in isolated silos. Each task is tagged `[M365]` `[AZ-104]` `[SC-200]` so it's clear which certification it supports (a task can carry more than one tag).

**Environment:** `BOL959.onmicrosoft.com` | Azure ($200 credit) + Business Premium + Entra ID P2 + EMS E5 + Defender for O365 P2 + Sentinel

⚠️ **Most important rule:** always **Stop (Deallocate)** or **Delete** Azure resources like VMs/Storage after use — otherwise the $200 credit burns through quickly.

---

## Project 3 — Identity Security (MFA, Conditional Access)
`[M365]` `[SC-200]` `[AZ-104]`

- [ ] Enable per-user MFA and test-login to verify it
- [ ] Conditional Access policy — require MFA for a specific group
- [ ] Build a location-based CA policy
- [ ] Set up Sign-in Risk Policy and User Risk Policy (Identity Protection)
- [ ] Deliberately fail a login 5–6 times for one user (raw material to hunt for in Sentinel later)

---

## Project 4 — Admin Roles & RBAC (both M365 and Azure)
`[M365]` `[AZ-104]`

- [ ] M365: assign a user the "User Administrator" role and verify what it grants
- [ ] M365: create an Administrative Unit for scoped admin access
- [ ] M365: set up "eligible" (just-in-time) role assignment via PIM
- [ ] `[AZ-104]` Azure RBAC: assign a user the "Reader" role on a Resource Group, sign in as that user and verify the access
- [ ] `[AZ-104]` create a custom RBAC role (e.g. can only start/stop a VM, not delete it)
- [ ] `[AZ-104]` apply a Resource Lock (Delete lock) to practice protecting a critical resource from accidental deletion

---

## Project 5 — Azure Storage
`[AZ-104]`
*Why here: identity and governance are in place — now it's time to build somewhere to put data, which is easier to understand before VMs/networking.*

- [ ] Create a Storage Account (Standard, LRS)
- [ ] Create a Blob container and upload/download a file
- [ ] Change access tier (Hot/Cool/Archive) and understand the difference
- [ ] Create an Azure File Share and map it as a drive
- [ ] Generate a Shared Access Signature (SAS) token for temporary access
- [ ] Build a lifecycle management policy (blob auto-moves to Cool tier after 30 days)
- [ ] Enable soft delete, delete a file, and recover it

---

## Project 6 — Azure Networking
`[AZ-104]`

- [ ] Create a Virtual Network (VNet) with 2 subnets
- [ ] Build a Network Security Group (NSG) with inbound/outbound rules (e.g. allow RDP/SSH only from your own IP)
- [ ] Create and associate a Public IP
- [ ] Set up VNet Peering between two VNets
- [ ] Explore Azure Bastion or just-in-time VM access (secure remote access)
- [ ] Create and explore a private DNS zone in Azure DNS

---

## Project 7 — Azure Compute (VM) — the biggest opportunity is hidden here
`[AZ-104]` `[SC-200]`
*Why this matters: this VM isn't just for AZ-104 — installing Windows Server + Active Directory Domain Services (AD DS) on it gives you a real on-prem-style domain controller, which turns Defender for Identity from a UI-only tour into a source of real data!*

- [ ] Create a Windows Server VM (B-series burstable — cheapest option)
- [ ] Create a Linux VM (Ubuntu) — practice the difference between RDP and SSH access
- [ ] Resize a VM (B1s → B2s), then scale back down
- [ ] Attach a Managed Disk and take a snapshot
- [ ] Install a VM Extension (e.g. Custom Script Extension)
- [ ] Configure an Availability Set or Availability Zone (high-availability concept)
- [ ] **Install the Active Directory Domain Services (AD DS) role on the Windows VM** — this becomes your own mini on-prem AD
- [ ] Install the Microsoft Defender for Identity sensor on that AD DS instance (the sensor download link is under Defender portal → Settings → Identities)
- [ ] **Stop (Deallocate)** the VM when done — leaving it running burns through credit fast

---

## Project 8 — Building and protecting the mailroom
`[M365]` `[SC-200]`

- [ ] Explore the Exchange admin center, create a shared mailbox
- [ ] Build a mail flow rule
- [ ] Trace a message's delivery path with Message Trace
- [ ] Customize anti-phishing, anti-spam, and anti-malware policies
- [ ] Test anti-malware detection with an EICAR test file
- [ ] Test Safe Links (between your own two email addresses)
- [ ] Review/release/delete a message from the Quarantine page
- [ ] Run **Attack Simulation Training** for a real phishing simulation

---

## Project 9 — Devices & Endpoints
`[M365]` `[SC-200]`

- [ ] Explore the Intune admin center
- [ ] Build a Compliance Policy and a Configuration Profile
- [ ] Enroll a device if possible (the Project 7 Azure VM can also be onboarded to Defender for Endpoint)
- [ ] Test endpoint detection with the EICAR file
- [ ] Explore the Exposure/Vulnerability Management section

---

## Project 10 — Files, Collaboration & External Apps
`[M365]` `[SC-200]`

- [ ] Create a SharePoint site collection, toggle external sharing
- [ ] Create a Team and test meeting/external-access policies
- [ ] Explore Defender for Cloud Apps → app catalog, risk scores
- [ ] Build and test a File policy

---

## Project 11 — Data Protection & Compliance
`[M365]`

- [ ] Explore the Microsoft Purview compliance portal
- [ ] Build a Retention Policy
- [ ] Build a basic DLP policy
- [ ] Search the Audit Log to trace all prior activity

---

## Project 12 — Azure Monitor + Log Analytics + Sentinel Integration
`[AZ-104]` `[SC-200]`
*Why together: AZ-104's "Monitor and maintain Azure resources" domain and Sentinel's foundation (Log Analytics workspace) are the same underlying technology — learning both together is the most efficient path.*

- [ ] Install the Azure Monitor Agent on the Project 7 VM to send diagnostics data
- [ ] Set up Metrics and Alerts (e.g. email alert if CPU exceeds 80%)
- [ ] Review the VM's Activity Log (who did what to the resource)
- [ ] Walk through every connector's "Open connector page" on the Data Connectors page
- [ ] Browse the `SigninLogs`, `AuditLogs`, `SecurityAlert`, and `Heartbeat` (from the VM) tables in Log Analytics

---

## Project 13 — KQL (Kusto Query Language)
`[SC-200]` `[AZ-104]`

- [ ] Basic syntax: `where`, `project`, `summarize`, `sort by`
- [ ] Find the failed login attempts from Project 3 in `SigninLogs`
- [ ] Analyze with `summarize count() by UserPrincipalName`
- [ ] Practice joining two tables with the `join` operator
- [ ] Use a time filter (`ago(24h)`)
- [ ] Query the VM's `Heartbeat`/`Perf` tables to check resource health
- [ ] Launch a built-in Workbook (Identity & Access)

---

## Project 14 — Analytics Rules (Detection Engineering)
`[SC-200]`

- [ ] Enable a built-in Analytics Rule from the Content Hub
- [ ] Build your own Scheduled Query Rule (based on the fail-login pattern)
- [ ] Set severity and MITRE ATT&CK tactic mapping
- [ ] Review the Incident generated by the rule

---

## Project 15 — Incident Investigation (now with real Identity data)
`[SC-200]`
*This is where the payoff from installing AD DS + the Defender for Identity sensor in Project 7 shows up — the Identities section now has real data.*

- [ ] Open an incident and review evidence, entities, and timeline
- [ ] Fully investigate the phishing incident generated by the Attack Simulation
- [ ] Investigate any Defender for Identity alert (simulate a suspicious login on the VM)
- [ ] Browse built-in queries in the Hunting section and run a custom hunting query
- [ ] Create a bookmark

---

## Project 16 — Automation, Backup, Cost Management & Reporting
`[M365]` `[AZ-104]` `[SC-200]`

- [ ] `[SC-200]` Build a Playbook (email notification when a new incident arrives)
- [ ] `[SC-200]` Use an Automation Rule to auto-trigger the playbook from an Analytics Rule
- [ ] `[AZ-104]` Set up Azure Backup and take a backup of the Project 7 VM
- [ ] `[AZ-104]` Review spend by resource in Cost Management + Billing, and set a budget alert
- [ ] `[M365]` Check Usage Reports and Secure Score, implement 2–3 recommendations
- [ ] `[AZ-104]` Delete all unused resources (VM, disk, public IP) at the end of the session to conserve credit

---

## How to proceed

1. Follow the sequence — each project builds the foundation or data the next one needs (the Project 7 AD DS VM and the Project 3 failed logins are both used again later)
2. Make it a habit to **always stop/delete** Azure resources (VMs especially) after use
3. Keep 2–3 lines of notes at the end of every project
4. Rough timeline: Projects 3–4 (1 week), Projects 5–6 (1.5–2 weeks, Azure-heavy), Projects 8–10 (2 weeks), Projects 12–15 (2–3 weeks)
5. In roughly 7–8 weeks, the hands-on foundation for all three certifications should be in place
