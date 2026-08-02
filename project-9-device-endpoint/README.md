# Project 9 — Device & Endpoint (Intune + Microsoft Defender for Endpoint)

**Goal:** stand up an Intune baseline for Windows devices (compliance + configuration), onboard the Project 7 domain controller VM into Microsoft Defender for Endpoint, and explore the Exposure Management / Vulnerability Management dashboards — all on free-tier M365 + Defender for Endpoint Plan 2 trial licensing.

**Certification focus:** SC-200 / AZ-104

---

## Why this is a long post

Most Intune walkthroughs show a clean policy created in three clicks. This one shows a profile type dropdown that silently accepted a category header instead of a real template, a compliance toggle that stayed greyed out until its parent setting was enabled first, a settings-catalog checkbox that added a setting without turning its value on, an onboarding package that looked like it had downloaded to the wrong machine, and a freshly onboarded device that showed up twice in Device Inventory. All of it is left in, because that's what actually happens the first time through.

---

## What was built

| # | Item | Result |
| --- | --- | --- |
| 1 | Intune admin center | Explored Home dashboard, Devices overview, and left-nav structure |
| 2 | Compliance policy | `CP-Windows10-Baseline` — BitLocker, Firewall, Antivirus, Real-time protection all Required, assigned to All devices |
| 3 | Configuration profile | `CFG-Windows10-SecurityBaseline` (Settings catalog) — BitLocker device encryption + Firewall enabled on Domain/Private/Public profiles, assigned to All devices |
| 4 | Device enrollment | `vm-win-addc` (Project 7's domain controller) onboarded to Microsoft Defender for Endpoint via local script, detection test run |
| 5 | EICAR endpoint test | Consciously skipped — malware detection already proven at the mail layer in Project 8 |
| 6 | Exposure management | Explored Exposure Management Overview and Vulnerability Management (Endpoint tab) dashboards |

All policies were assigned to **All devices** rather than a scoped group, matching this lab's single-admin, single-device-so-far scale.

---

## 9.1 — Intune Admin Center Explore


Starting point: signing into `intune.microsoft.com` with the `rakibulhcrakib2` admin account for the first time this project.

Microsoft Intune admin center home dashboard after signing in with the rakibulhcrakib2 admin account, showing the Status panel (0 devices not in compliance, 0 configuration errors) and the Mandatory Azure MFA Phase 2 platform-wide notification.

[![Microsoft Intune admin center home dashboard after signing i](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/01.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/01.png)

Devices | Overview page showing device counts by platform (all at 0, since no devices were enrolled yet) and the left navigation exposing Configuration, Compliance, and Conditional access under Manage devices.

[![Devices | Overview page showing device counts by platform (a](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/02.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/02.png)

---

## 9.2 — Compliance Policy & Configuration Profile (Windows 10/11 Baseline)


Building a paired Compliance Policy (evaluation) and Configuration Profile (enforcement) for the same four core controls — BitLocker, Firewall, Antivirus, Real-time protection.

Devices | Compliance policies list (empty) with the Create a policy panel open, selecting Windows 10 and later as the target platform.

[![Devices | Compliance policies list (empty) with the Create a](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/03.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/03.png)

> ⚠️ **Mistake: the Profile type dropdown accidentally selected 'Templates' - a category header, not an actual selectable profile - leaving the panel showing only a search box with no data.**

[![Mistake: the Profile type dropdown accidentally selected 'Te](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/04.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/04.png)

The resulting 'No data' state confirmed the wrong item had been picked, since Templates itself carries no compliance settings to configure.

[![The resulting 'No data' state confirmed the wrong item had b](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/05.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/05.png)

**Fix applied: re-opening the Profile type dropdown and selecting 'Windows 10/11 compliance policy' directly, the only real selectable template for this platform.**

[![Fix applied: re-opening the Profile type dropdown and select](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/06.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/06.png)

The five-step policy wizard (Basics, Compliance settings, Actions for noncompliance, Assignments, Review + create) opening on the Basics tab with Platform and Profile type locked in as read-only.

[![The five-step policy wizard (Basics, Compliance settings, Ac](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/07.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/07.png)

Name field filled in as CP-Windows10-Baseline, confirmed valid with a green checkmark, following the lab's established naming convention.

[![Name field filled in as CP-Windows10-Baseline, confirmed val](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/08.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/08.png)

Compliance settings tab opened, showing Custom Compliance, Device Health (BitLocker, Secure Boot, Code integrity) and Device Properties categories, all defaulted to Not configured.

[![Compliance settings tab opened, showing Custom Compliance, D](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/09.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/09.png)

System Security settings including Password policy, Encryption, Device Security (Firewall, TPM, Antivirus, Antispyware) and the start of the Defender category, all still Not configured at this point.

[![System Security settings including Password policy, Encrypti](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/10.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/10.png)

Bottom of the Defender section including Microsoft Defender for Endpoint machine risk score integration, plus the Windows Subsystem for Linux (WSL) settings block.

[![Bottom of the Defender section including Microsoft Defender ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/11.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/11.png)

BitLocker and Firewall toggles successfully switched to Require (shown in purple), while Antivirus and Real-time protection were still left at Not configured at this point in the walkthrough.

[![BitLocker and Firewall toggles successfully switched to Requ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/12.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/12.png)

Antivirus toggled to Require, but the Real-time protection toggle underneath Microsoft Defender Antimalware appeared greyed out and unclickable, prompting a 'this can't be done' question.

[![Antivirus toggled to Require, but the Real-time protection t](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/13.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/13.png)

**Fix: enabling the parent 'Microsoft Defender Antimalware' setting to Require unlocked the child 'Real-time protection' toggle, which was then also set to Require - completing all four core baseline controls.**

[![Fix: enabling the parent 'Microsoft Defender Antimalware' se](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/14.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/14.png)

Actions for noncompliance tab showing the default, non-removable action ('Mark device noncompliant', scheduled Immediately) with an optional empty row for a second action, left unused.

[![Actions for noncompliance tab showing the default, non-remov](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/15.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/15.png)

Assignments tab with Included groups and Excluded groups both empty, offering Add groups, Add all users, and Add all devices as assignment targets.

[![Assignments tab with Included groups and Excluded groups bot](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/16.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/16.png)

Assignments confirmed with 'All devices' set to Active status, followed by the newly created CP-Windows10-Baseline policy's own Monitor page showing 0 compliant / 0 noncompliant / 0 total devices (expected, since no device was enrolled yet).

[![Assignments confirmed with 'All devices' set to Active statu](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/17.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/17.png)

CP-Windows10-Baseline policy detail page confirming successful creation, with Monitor and Properties tabs and a Per-setting status report link available.

[![CP-Windows10-Baseline policy detail page confirming successf](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/18.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/18.png)

---

## 9.2 — Compliance Policy & Configuration Profile (Windows 10/11 Baseline)


Building a paired Compliance Policy (evaluation) and Configuration Profile (enforcement) for the same four core controls — BitLocker, Firewall, Antivirus, Real-time protection.

Devices | Configuration policies list already showing two Microsoft-provided default profiles (Firewall Windows default policy, NGP Windows default policy) auto-created by the tenant, with the Create dropdown open to start a new policy.

[![Devices | Configuration policies list already showing two Mi](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/19.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/19.png)

Create a profile panel with Platform and Profile type dropdowns both empty, about to be set to Windows 10 and later / Settings catalog.

[![Create a profile panel with Platform and Profile type dropdo](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/20.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/20.png)

Windows 10 and later platform paired with Settings catalog profile type selected, described as 'start from scratch and select settings you want from the library of available settings'.

[![Windows 10 and later platform paired with Settings catalog p](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/21.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/21.png)

Five-step Create profile wizard (Basics, Configuration settings, Scope tags, Assignments, Review + create) opening on Basics with Name, Description, and read-only Platform fields.

[![Five-step Create profile wizard (Basics, Configuration setti](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/22.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/22.png)

Name field completed as CFG-Windows10-SecurityBaseline with a green checkmark, matching the naming convention established for the compliance policy.

[![Name field completed as CFG-Windows10-SecurityBaseline with ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/23.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/23.png)

Configuration settings tab in its empty starting state, showing the Settings catalog explanation and the '+ Add settings' entry point into the settings picker.

[![Configuration settings tab in its empty starting state, show](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/24.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/24.png)

Settings picker searched for 'Firewall', showing the 74-setting Firewall category with 'Enable Domain Network Firewall' already checked as the first of three network profile toggles needed.

[![Settings picker searched for 'Firewall', showing the 74-sett](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/25.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/25.png)

Enable Private Network Firewall checked, automatically pulling in roughly twenty related child settings (IPsec policy merge, default inbound/outbound action, stealth mode, logging) at their sensible default values.

[![Enable Private Network Firewall checked, automatically pulli](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/26.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/26.png)

All three network firewall profiles (Domain, Private, Public) confirmed checked in the settings picker, with 51 of 74 Firewall settings now configured in the main panel.

[![All three network firewall profiles (Domain, Private, Public](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/27.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/27.png)

Settings picker searched for 'BitLocker', showing only 5 settings in this category, with 'Allow Warning For Other Disk Encryption' accidentally pre-checked instead of the intended 'Require Device Encryption'.

[![Settings picker searched for 'BitLocker', showing only 5 set](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/28.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/28.png)

> ⚠️ **Mistake: after the correction attempt, 'Require Device Encryption' still showed as Disabled while an unwanted 'Configure Recovery Password Rotation' setting had been added, since checking an item only adds it - the value still needs a separate toggle.**

[![Mistake: after the correction attempt, 'Require Device Encry](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/29.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/29.png)

**Fix applied: Require Device Encryption toggled to Enabled, while the two unwanted BitLocker settings were removed via the minus icon, leaving BitLocker correctly configured with just the one required setting.**

[![Fix applied: Require Device Encryption toggled to Enabled, w](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/30.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/30.png)

Scope tags tab showing the Default tag already applied, left unchanged for this single-admin lab environment.

[![Scope tags tab showing the Default tag already applied, left](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/31.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/31.png)

Assignments tab for the configuration profile in its empty state, mirroring the same Add groups / Add all users / Add all devices options seen for the compliance policy.

[![Assignments tab for the configuration profile in its empty s](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/32.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/32.png)

Review + create summary page confirming CFG-Windows10-SecurityBaseline's Basics, BitLocker and Firewall configuration settings, Default scope tag, and All devices assignment - ready to submit.

[![Review + create summary page confirming CFG-Windows10-Securi](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/33.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/33.png)

Configuration policies list confirming CFG-Windows10-SecurityBaseline created successfully (Settings catalog type, Default scope tag), alongside the two pre-existing Microsoft default policies, completing Task 2.

[![Configuration policies list confirming CFG-Windows10-Securit](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/34.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/34.png)

---

## 9.3 — Device Enrollment: Onboarding vm-win-addc to Defender for Endpoint


`vm-win-addc` — the Windows Server 2022 domain controller built in Project 7 — restarted, reached via a freshly re-created Azure Bastion, and onboarded into Microsoft Defender for Endpoint using the local script method.

Azure Portal showing vm-win-addc restarted to a Running state (Standard D2s v7, 2 vCPUs) ahead of onboarding it to Microsoft Defender for Endpoint, reusing the Windows Server Domain Controller VM built in Project 7.

[![Azure Portal showing vm-win-addc restarted to a Running stat](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/35.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/35.png)

A new Azure Bastion resource (vnet-lab-sc200-bastion) successfully provisioned in the AzureBastionSubnet, re-created after the original Project 6 Bastion had been cleaned up, chosen over direct RDP to avoid the dynamic-IP NSG problem.

[![A new Azure Bastion resource (vnet-lab-sc200-bastion) succes](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/36.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/36.png)

Successful browser-based Bastion RDP session into vm-win-addc's desktop, with Server Manager open showing 1 local server and 1 server total under Roles and Server Groups.

[![Successful browser-based Bastion RDP session into vm-win-add](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/37.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/37.png)

Microsoft Defender portal's Endpoints Onboarding page, reached from inside the VM's own Edge browser, showing Intune, Defender for Cloud, Defender deployment tool, Mobile device management, and Configuration Manager deployment options.

[![Microsoft Defender portal's Endpoints Onboarding page, reach](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/38.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/38.png)

> ⚠️ **Mistake: Step 1 initially left on 'Windows 10 and 11' as the operating system, which is the wrong package type for a Windows Server 2022 domain controller VM.**

[![Mistake: Step 1 initially left on 'Windows 10 and 11' as the](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/39.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/39.png)

**Fix: Step 1 corrected to 'Windows Server 2019, 2022, and 2025', with Deployment method set to 'Local Script (for up to 10 devices)' - the right combination for a single Windows Server lab VM.**

[![Fix: Step 1 corrected to 'Windows Server 2019, 2022, and 202](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/40.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/40.png)

A File Explorer window showing a 'Project 09' folder containing the downloaded GatewayWindowsDefenderATPOnboardingPackage file - initially mistaken for the researcher's own local PC rather than the VM, since the folder names matched the lab's personal project structure.

[![A File Explorer window showing a 'Project 09' folder contain](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/41.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/41.png)

Confirmation that the onboarding package had in fact downloaded inside the VM itself: the Edge Downloads panel next to the VM's own Recycle Bin and Microsoft Edge desktop icons, resolving the earlier local-PC-vs-VM confusion without needing the planned SAS token transfer.

[![Confirmation that the onboarding package had in fact downloa](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/42.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/42.png)

The extracted onboarding script (WindowsDefenderATPOnboardingScript.cmd) run directly inside an elevated Command Prompt, completing with 'Successfully onboarded machine to Microsoft Defender for Endpoint'.

[![The extracted onboarding script (WindowsDefenderATPOnboardin](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/43.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/43.png)

The official Microsoft detection-test PowerShell command pasted into a fresh elevated Command Prompt, about to be run to trigger a harmless simulated malicious file download and confirm the sensor is actively reporting.

[![The official Microsoft detection-test PowerShell command pas](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/44.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/44.png)

Device Inventory in the Defender portal showing vm-win-addc.rakibul.local listed twice - one entry correctly marked Onboarded (from the sensor script) and a second marked 'Insufficient info' (likely from passive AD/domain discovery) - a duplicate-entry finding expected to self-resolve as Microsoft's deduplication logic catches up.

[![Device Inventory in the Defender portal showing vm-win-addc.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/45.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/45.png)

---

## 9.4 — Exposure & Vulnerability Management Explore


A first look at Exposure Management and Vulnerability Management immediately after onboarding — useful for documenting what a brand-new, unscanned device actually looks like in these dashboards.

Exposure management left navigation expanded, showing Overview, Secure now, Initiatives, Recommendations, Vulnerability management, Attack surface, Secure score, and Data connectors.

[![Exposure management left navigation expanded, showing Overvi](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/46.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/46.png)

Exposure Management Overview dashboard: Patch and Mitigate cards showing temporary load errors (expected right after onboarding, before the backend fully processes the new device), a Fix card reporting 'No critical issues', and Domain initiatives showing Endpoint at 0% versus Identity at 70% and SaaS at 45% from earlier projects' data.

[![Exposure Management Overview dashboard: Patch and Mitigate c](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/47.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/47.png)

Vulnerability management Endpoint tab showing a 0/100 exposure score and 'No data' across all charts (Top impactful events, Top vulnerable software, Top exposed devices) - the expected baseline state for a device onboarded only minutes earlier, since the first full vulnerability scan takes hours to complete.

[![Vulnerability management Endpoint tab showing a 0/100 exposu](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-9-device-endpoint/images/48.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-9-device-endpoint/images/48.png)

---

## Mistakes & fixes (quick reference)

| Mistake | Fix |
| --- | --- |
| Compliance policy Profile type dropdown accepted 'Templates' (a category header) instead of a real template | Re-opened the dropdown and selected 'Windows 10/11 compliance policy' directly |
| Real-time protection toggle stayed greyed out | Enabled the parent 'Microsoft Defender Antimalware' setting first, which unlocked the child setting |
| Configuration profile: checking a BitLocker setting in the picker added it but left its value at the default, not 'Enabled' | Learned that checking a setting only adds it to the profile — the value still needs a separate toggle |
| Onboarding package appeared to download to the researcher's own local PC, not the VM | Confirmed via the in-VM Edge Downloads panel that it had, in fact, downloaded inside the VM all along |
| `vm-win-addc` showed up twice in Device Inventory after onboarding (Onboarded + Insufficient info) | Left as-is — expected to self-resolve as Microsoft's device deduplication logic catches up over time |
| Defender for Endpoint deployment package initially left on 'Windows 10 and 11' instead of the correct Server OS | Reselected 'Windows Server 2019, 2022, and 2025' before downloading the onboarding package |

---

## Key learnings

1. **Compliance Policy vs. Configuration Profile are two different jobs** — one evaluates a device's state, the other pushes settings to enforce that state; a real baseline needs both.
2. **Settings Catalog auto-pulls related child settings** when a parent setting is checked, at sensible defaults — but checking a setting is not the same as turning its value on.
3. **Some settings have hard dependencies on their parent** — Real-time protection can't be configured until Microsoft Defender Antimalware itself is set to Require.
4. **A tenant ships with default configuration profiles already in place** (Firewall Windows default policy, NGP Windows default policy) before any admin creates one manually.
5. **Onboarding-package OS selection matters** — a Windows Server VM needs the Server-specific package, not the Windows 10/11 one.
6. **Freshly onboarded devices can appear as duplicate entries** in Device Inventory (sensor-based vs. domain-discovery-based) until Microsoft's backend deduplicates them.
7. **Exposure Management and Vulnerability Management need time to populate** — a device onboarded minutes earlier shows a 0/100 exposure score and 'No data' across every chart, which is expected, not broken.
8. **Not every test needs repeating per-project** — EICAR malware detection was proven once at the mail layer in Project 8; skipping the endpoint-layer repeat was a deliberate scope decision, documented rather than silently omitted.

---

**Next:** Project 10 — Collaboration Security (SharePoint, Teams, Defender for Cloud Apps). Full roadmap: [`ROADMAP.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/ROADMAP.md)
