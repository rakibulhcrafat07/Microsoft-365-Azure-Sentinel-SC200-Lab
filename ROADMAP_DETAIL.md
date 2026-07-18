# Roadmap Detail — Project 3 থেকে Project 16 (M365 Admin + SC-200 + AZ-104)

তিনটা সার্টিফিকেশন ট্র্যাক (M365 Admin, SC-200, AZ-104) আলাদা না রেখে **একটাই বাস্তবসম্মত ক্রমে** সাজানো হয়েছে — যেভাবে একটা কোম্পানির IT ইনফ্রাস্ট্রাকচার, ইউজার ম্যানেজমেন্ট আর সিকিউরিটি একসাথে গড়ে ওঠে। প্রতিটা টাস্কের পাশে ট্যাগ দেওয়া আছে `[M365]` `[AZ-104]` `[SC-200]` — যাতে বুঝতে পারো কোনটা কোন সার্টিফিকেশনের জন্য কাজে লাগছে (একটা টাস্ক একাধিক ট্যাগও পেতে পারে)।

**Environment:** BOL959.onmicrosoft.com | Azure ($200 credit) + Business Premium + Entra ID P2 + EMS E5 + Defender for O365 P2 + Sentinel

⚠️ **সবচেয়ে জরুরি নিয়ম:** VM/Storage-এর মতো Azure রিসোর্স ব্যবহারের পর সবসময় **Stop (Deallocate)** বা **Delete** করে দেবে, নাহলে $200 ক্রেডিট দ্রুত ফুরিয়ে যাবে।

---

## Project 3 — Identity Security (MFA, Conditional Access)
`[M365]` `[SC-200]` `[AZ-104]`

- [ ] MFA per-user enable করে টেস্ট লগইন করা
- [ ] Conditional Access policy — নির্দিষ্ট group-এর জন্য MFA বাধ্যতামূলক
- [ ] Location-based CA policy বানানো
- [ ] Sign-in risk policy ও User risk policy (Identity Protection) সেটাপ করা
- [ ] একটা user ইচ্ছাকৃতভাবে ৫-৬ বার ভুল পাসওয়ার্ড দিয়ে লগইন attempt করা (পরে Sentinel-এ খুঁজে বের করার জন্য কাঁচামাল)

---

## Project 4 — Admin Roles ও RBAC (M365 + Azure দুই জায়গাতেই)
`[M365]` `[AZ-104]`

- [ ] M365: একজন user-কে "User Administrator" role দিয়ে ক্ষমতা যাচাই করা
- [ ] M365: Administrative Unit বানিয়ে scoped admin access
- [ ] M365: PIM দিয়ে "eligible" role assignment (just-in-time admin) সেটাপ
- [ ] `[AZ-104]` Azure RBAC: একটা Resource Group-এ একজন user-কে "Reader" role assign করা, সেই user দিয়ে লগইন করে ক্ষমতা যাচাই করা
- [ ] `[AZ-104]` একটা custom RBAC role বানানো (যেমন শুধু VM start/stop করতে পারবে, delete না)
- [ ] `[AZ-104]` একটা Resource Lock (Delete lock) বসিয়ে গুরুত্বপূর্ণ রিসোর্স accidental delete থেকে বাঁচানোর practice

---

## Project 5 — Azure Storage
`[AZ-104]`
*কেন এখানে: ইউজার ও গভর্নেন্স রেডি, এখন ডেটা রাখার জায়গা বানানোর পালা — এটা VM/networking-এর আগে করলে বুঝতে সহজ হয়।*

- [ ] একটা Storage Account বানানো (Standard, LRS)
- [ ] Blob container বানিয়ে ফাইল upload/download করা
- [ ] Access tier (Hot/Cool/Archive) পরিবর্তন করে পার্থক্য বোঝা
- [ ] Azure File Share বানিয়ে map drive করে দেখা
- [ ] Shared Access Signature (SAS) টোকেন জেনারেট করে সাময়িক অ্যাক্সেস দেওয়া
- [ ] Lifecycle management policy বানানো (৩০ দিন পর blob automatically Cool tier-এ যাবে)
- [ ] Soft delete enable করে একটা ফাইল delete করে recover করা

---

## Project 6 — Azure Networking
`[AZ-104]`

- [ ] একটা Virtual Network (VNet) বানানো, ২টা subnet সহ
- [ ] Network Security Group (NSG) বানিয়ে inbound/outbound rule সেট করা (যেমন শুধু নিজের IP থেকে RDP/SSH allow)
- [ ] একটা Public IP বানানো ও associate করা
- [ ] দুইটা VNet বানিয়ে VNet Peering সেটাপ করা
- [ ] Azure Bastion বা just-in-time VM access explore করা (নিরাপদ remote access)
- [ ] Azure DNS-এ একটা private DNS zone বানিয়ে explore করা

---

## Project 7 — Azure Compute (VM) — এখানেই বড় সুযোগ লুকিয়ে আছে
`[AZ-104]` `[SC-200]`
*কেন এখানে গুরুত্বপূর্ণ: এই VM শুধু AZ-104-এর জন্য না — এটাতে Windows Server + Active Directory Domain Services (AD DS) বসালে তুমি একটা বাস্তব on-prem-স্টাইল ডোমেইন কন্ট্রোলার পাবে, যেটা দিয়ে Defender for Identity-তে আগে যেটা শুধু UI দেখা যাচ্ছিল, এখন থেকে বাস্তব ডেটা পাবে!*

- [ ] একটা Windows Server VM বানানো (B-series burstable, সবচেয়ে সস্তা)
- [ ] একটা Linux VM বানানো (Ubuntu) — RDP vs SSH access-এর পার্থক্য practice করা
- [ ] VM resize করা (B1s → B2s), তারপর আবার ছোট করা
- [ ] Managed Disk যোগ করা, snapshot নেওয়া
- [ ] VM Extension ইনস্টল করা (যেমন Custom Script Extension)
- [ ] Availability Set বা Availability Zone কনফিগার করে বোঝা (high availability concept)
- [ ] **Windows VM-এ Active Directory Domain Services (AD DS) role ইনস্টল করা** — এটাই তোমার নিজের mini on-prem AD
- [ ] সেই AD DS-এ Microsoft Defender for Identity sensor ইনস্টল করা (Defender portal → Settings → Identities থেকে sensor download লিংক পাওয়া যায়)
- [ ] কাজ শেষে VM **Stop (Deallocate)** করা — চালু রাখলে ক্রেডিট দ্রুত শেষ হবে

---

## Project 8 — মেইলরুম বানানো ও প্রোটেক্ট করা
`[M365]` `[SC-200]`

- [ ] Exchange admin center explore করা, একটা shared mailbox বানানো
- [ ] একটা mail flow rule বানানো
- [ ] Message trace দিয়ে delivery path ট্র্যাক করা
- [ ] Anti-phishing, anti-spam, anti-malware policy কাস্টমাইজ করা
- [ ] EICAR টেস্ট ফাইল দিয়ে anti-malware detection টেস্ট করা
- [ ] Safe Links টেস্ট (নিজের দুই ইমেইলের মধ্যে)
- [ ] Quarantine পেজে গিয়ে review/release/delete করা
- [ ] **Attack Simulation Training** চালিয়ে বাস্তব ফিশিং সিমুলেশন রান করা

---

## Project 9 — ডিভাইস ও এন্ডপয়েন্ট
`[M365]` `[SC-200]`

- [ ] Intune admin center explore করা
- [ ] Compliance Policy ও Configuration Profile বানানো
- [ ] সম্ভব হলে একটা ডিভাইস enroll করা (Project 7-এর Azure VM-ও Defender for Endpoint-এ onboard করা যায়)
- [ ] EICAR ফাইল দিয়ে endpoint detection টেস্ট
- [ ] Exposure/Vulnerability management সেকশন explore করা

---

## Project 10 — ফাইল, কোলাবরেশন ও বাইরের অ্যাপ
`[M365]` `[SC-200]`

- [ ] SharePoint site collection বানানো, external sharing টগল করা
- [ ] Teams বানিয়ে meeting/external access policy টেস্ট করা
- [ ] Defender for Cloud Apps → app catalog, risk score explore করা
- [ ] একটা File policy বানিয়ে টেস্ট করা

---

## Project 11 — ডেটা প্রোটেকশন ও কমপ্লায়েন্স
`[M365]`

- [ ] Microsoft Purview compliance portal explore করা
- [ ] Retention Policy বানানো
- [ ] Basic DLP policy বানানো
- [ ] Audit log search করে আগের সব activity ট্র্যাক করা

---

## Project 12 — Azure Monitor + Log Analytics + Sentinel সংযোগ
`[AZ-104]` `[SC-200]`
*কেন একসাথে: AZ-104-এর "Monitor and maintain Azure resources" ডোমেইন আর Sentinel-এর ভিত্তি (Log Analytics workspace) একই টেকনোলজি — এখানে দুটো একসাথে শেখা সবচেয়ে efficient।*

- [ ] Project 7-এর VM-এ Azure Monitor Agent ইনস্টল করে diagnostics ডেটা পাঠানো
- [ ] Metrics ও Alerts (যেমন CPU 80%-এর বেশি হলে email alert) সেটাপ করা
- [ ] VM-এর Activity Log দেখা (কে কী করেছে রিসোর্সে)
- [ ] Data connectors পেজে প্রতিটা connector-এর "Open connector page" ঘুরে দেখা
- [ ] Log Analytics-এ `SigninLogs`, `AuditLogs`, `SecurityAlert`, `Heartbeat` (VM থেকে) টেবিল browse করা

---

## Project 13 — KQL (Kusto Query Language)
`[SC-200]` `[AZ-104]`

- [ ] বেসিক syntax: `where`, `project`, `summarize`, `sort by`
- [ ] Project 3-এর ভুল পাসওয়ার্ড attempt-গুলো `SigninLogs`-এ খুঁজে বের করা
- [ ] `summarize count() by UserPrincipalName` দিয়ে বিশ্লেষণ করা
- [ ] `join` অপারেটর দিয়ে দুইটা টেবিল জোড়া লাগানো
- [ ] Time filter (`ago(24h)`) ব্যবহার করা
- [ ] VM-এর `Heartbeat`/`Perf` টেবিল কুয়েরি করে resource health দেখা
- [ ] একটা built-in Workbook (Identity & Access) চালু করা

---

## Project 14 — Analytics Rules (Detection Engineering)
`[SC-200]`

- [ ] Content Hub থেকে একটা built-in Analytics Rule enable করা
- [ ] নিজের Scheduled Query Rule বানানো (fail login pattern দিয়ে)
- [ ] Severity ও MITRE ATT&CK mapping সেট করা
- [ ] Rule থেকে তৈরি Incident চেক করা

---

## Project 15 — Incident Investigation (এবার বাস্তব Identity ডেটা সহ)
`[SC-200]`
*Project 7-এ AD DS + Defender for Identity sensor বসানোর সুফল এখানে পাবে — এবার Identities সেকশনে সত্যিকারের ডেটা আসবে।*

- [ ] Incidents পেজে গিয়ে evidence, entities, timeline দেখা
- [ ] Attack Simulation থেকে তৈরি phishing incident end-to-end investigate করা
- [ ] Defender for Identity-এর কোনো alert থাকলে সেটা investigate করা (VM-এ suspicious login simulate করে)
- [ ] Hunting সেকশনে built-in query browse করে custom hunting query চালানো
- [ ] একটা bookmark তৈরি করা

---

## Project 16 — Automation, Backup, Cost Management ও Reporting
`[M365]` `[AZ-104]` `[SC-200]`

- [ ] `[SC-200]` একটা Playbook বানানো (নতুন incident এলে email notification)
- [ ] `[SC-200]` Automation rule দিয়ে Analytics Rule-এর সাথে playbook auto-trigger
- [ ] `[AZ-104]` Azure Backup সেটাপ করে Project 7-এর VM-এর একটা backup নেওয়া
- [ ] `[AZ-104]` Cost Management + Billing-এ গিয়ে কোন রিসোর্স কত খরচ করছে দেখা, budget alert সেট করা
- [ ] `[M365]` Usage reports ও Secure Score চেক করে ২-৩টা recommendation implement করা
- [ ] `[AZ-104]` সেশন শেষে অব্যবহৃত সব রিসোর্স (VM, disk, public IP) delete করে ক্রেডিট বাঁচানো

---

## এগোনোর নিয়ম

1. ক্রম মেনে চলো — প্রতিটা Phase পরেরটার জন্য ভিত্তি বা ডেটা তৈরি করে (বিশেষ করে Project 7-এর AD DS VM এবং Project 3-এর ভুল লগইন — এগুলো পরে কাজে লাগবে)
2. Azure রিসোর্স (VM বিশেষত) ব্যবহারের পর **সবসময় বন্ধ/ডিলিট** করার অভ্যাস করো
3. প্রতিটা Phase শেষে ২-৩ লাইন নোট রাখো
4. আনুমানিক সময়: Project 2-3 (১ সপ্তাহ), Project 5-6 (১.৫-২ সপ্তাহ, Azure-heavy), Project 8-10 (২ সপ্তাহ), Project 12-15 (২-৩ সপ্তাহ)
5. মোট প্রায় ৭-৮ সপ্তাহে তিনটা সার্টিফিকেশনের হ্যান্ডস-অন বেস তৈরি হয়ে যাবে
