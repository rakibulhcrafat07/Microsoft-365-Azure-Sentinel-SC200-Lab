# Project 8 - Mail Room: Build & Protect It (M365 + SC-200)

This project covers Exchange Online mail administration and Microsoft Defender for
Office 365 threat protection, built entirely on free-tier Microsoft 365 Business Premium
and Defender for Office 365 Plan 2 trial licensing in tenant `BOL959.onmicrosoft.com`.

Part of the **Zero-to-SOC: A Self-Funded, Multi-Certification Cloud Security Lab** series.

**Tags:** `M365` `SC-200`

## What was built

| # | Task | Result |
|---|------|--------|
| 1 | Shared mailbox | `IT Support` shared mailbox created, 2 delegates given Full Access |
| 2 | Mail flow rule | Subject-keyword rule that Bcc's admin, deployed in Audit (Test) mode |
| 3 | Message trace | Verified rule delivery + execution via message trace tooling |
| 4 | Threat policies | Custom anti-phishing, anti-spam, and anti-malware policies |
| 5 | EICAR test | Confirmed anti-malware quarantines a real (harmless) test virus |
| 6 | Safe Links | Custom Safe Links policy with real-time URL scanning |
| 7 | Attack simulation | Full credential-harvest phishing simulation, end to end |

## Repo structure
```
project-8-mail-protection/
  README.md          <- this file
  images/             <- numbered screenshots (01.png ... NN.png)
```


## Task 1 - Exchange Admin Center Explore & Shared Mailbox

![Exchange admin center home page after login](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/01.png)
*Exchange admin center home page after login*

![Mailboxes card with 'Add a shared mailbox' option](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/02.png)
*Mailboxes card with 'Add a shared mailbox' option*

![Add a shared mailbox form (Display name, Email address, Alias)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/03.png)
*Add a shared mailbox form (Display name, Email address, Alias)*

![Shared mailbox created successfully - IT Support mailbox](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/04.png)
*Shared mailbox created successfully - IT Support mailbox*

![Next step: Add users to this mailbox](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/05.png)
*Next step: Add users to this mailbox*

![Manage shared mailbox members panel (0 items)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/06.png)
*Manage shared mailbox members panel (0 items)*

![Full list of tenant users available to add as delegates](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/07.png)
*Full list of tenant users available to add as delegates*

![Md. Imran Ahmed and Pallob Kumar Roy selected for Full Access](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/08.png)
*Md. Imran Ahmed and Pallob Kumar Roy selected for Full Access*

![Shared mailbox members updated successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/09.png)
*Shared mailbox members updated successfully*


## Task 2 - Mail Flow Rule

![Mail flow > Rules page (empty, no rules yet)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/10.png)
*Mail flow > Rules page (empty, no rules yet)*

!['Add a rule' dropdown - choosing Create a new rule](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/11.png)
*'Add a rule' dropdown - choosing Create a new rule*

![New transport rule wizard - Set rule conditions step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/12.png)
*New transport rule wizard - Set rule conditions step*

!['Specify words or phrases' panel - adding keyword 'confidential'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/13.png)
*'Specify words or phrases' panel - adding keyword 'confidential'*

!['Do the following' dropdown showing available actions](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/14.png)
*'Do the following' dropdown showing available actions*

![Selecting 'to the Bcc box' as the recipient action](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/15.png)
*Selecting 'to the Bcc box' as the recipient action*

![Select members panel - full tenant recipient list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/16.png)
*Select members panel - full tenant recipient list*

![Admin account selected as Bcc recipient](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/17.png)
*Admin account selected as Bcc recipient*

![Rule condition and action confirmed before proceeding](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/18.png)
*Rule condition and action confirmed before proceeding*

![Set rule settings step - Rule mode options (Enforce / Test)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/19.png)
*Set rule settings step - Rule mode options (Enforce / Test)*

!['Test without Policy Tips' mode selected (Report-only discipline)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/20.png)
*'Test without Policy Tips' mode selected (Report-only discipline)*

![Review and finish - full rule summary before creation](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/21.png)
*Review and finish - full rule summary before creation*

![Transport rule created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/22.png)
*Transport rule created successfully*

![New rule appears Disabled by default in the Rules list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/23.png)
*New rule appears Disabled by default in the Rules list*

![Rule manually Enabled - status panel confirms Audit mode](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/24.png)
*Rule manually Enabled - status panel confirms Audit mode*


## Task 3 - Message Trace

![Composing test email with subject 'Confidential - Test Mail Trace'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/25.png)
*Composing test email with subject 'Confidential - Test Mail Trace'*

![Test email visible in Sent Items](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/26.png)
*Test email visible in Sent Items*

![New message trace search form with sender/recipient filters](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/27.png)
*New message trace search form with sender/recipient filters*

![Message trace filters confirmed (sender, recipient, time range)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/28.png)
*Message trace filters confirmed (sender, recipient, time range)*

![Message trace search results - status: Delivered](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/29.png)
*Message trace search results - status: Delivered*

![Detailed trace view - 'Message events: No data available'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/30.png)
*Detailed trace view - 'Message events: No data available'*

![Rules list - 'Last execution' column empty (delay observed)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/31.png)
*Rules list - 'Last execution' column empty (delay observed)*

![Rules list rechecked later - 'Last execution: 1 hour ago' confirmed](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/32.png)
*Rules list rechecked later - 'Last execution: 1 hour ago' confirmed*


## Task 4 - Anti-phishing, Anti-spam & Anti-malware Policies

![Microsoft Defender - Policies & rules landing page](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/33.png)
*Microsoft Defender - Policies & rules landing page*

![Threat policies overview - Anti-phishing, Anti-spam, Anti-malware, Safe Attachments, Safe Links](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/34.png)
*Threat policies overview - Anti-phishing, Anti-spam, Anti-malware, Safe Attachments, Safe Links*

![Anti-phishing policies list - default Office365 AntiPhish policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/35.png)
*Anti-phishing policies list - default Office365 AntiPhish policy*

![Create new anti-phishing policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/36.png)
*Create new anti-phishing policy - naming step*

![Users, groups and domains step - adding tenant domain](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/37.png)
*Users, groups and domains step - adding tenant domain*

![Domain BOL959.onmicrosoft.com added to policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/38.png)
*Domain BOL959.onmicrosoft.com added to policy scope*

![Phishing threshold & protection - Impersonation section](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/39.png)
*Phishing threshold & protection - Impersonation section*

!['Enable users to protect' checkbox not yet applied (first attempt)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/40.png)
*'Enable users to protect' checkbox not yet applied (first attempt)*

!['Enable users to protect' successfully checked](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/41.png)
*'Enable users to protect' successfully checked*

![Manage senders for impersonation protection panel](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/42.png)
*Manage senders for impersonation protection panel*

![Add user dialog - admin account added as protected sender](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/43.png)
*Add user dialog - admin account added as protected sender*

![Protected sender confirmed in the Add user list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/44.png)
*Protected sender confirmed in the Add user list*

![Session resumed - Anti-phishing list shows only default policy (draft lost)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/45.png)
*Session resumed - Anti-phishing list shows only default policy (draft lost)*

![Restarting wizard - Policy name step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/46.png)
*Restarting wizard - Policy name step*

![Policy named 'Executive Impersonation Protection'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/47.png)
*Policy named 'Executive Impersonation Protection'*

![Users, groups and domains step (second attempt)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/48.png)
*Users, groups and domains step (second attempt)*

![Tenant domain re-added to policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/49.png)
*Tenant domain re-added to policy scope*

![Phishing threshold & protection page reloaded](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/50.png)
*Phishing threshold & protection page reloaded*

![Enable users to protect checked, Manage senders link visible](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/51.png)
*Enable users to protect checked, Manage senders link visible*

![Manage senders panel - admin already listed as protected sender](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/52.png)
*Manage senders panel - admin already listed as protected sender*

![Sender confirmed before clicking Done](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/53.png)
*Sender confirmed before clicking Done*

![Actions page - impersonation action left as 'Don't apply any action'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/54.png)
*Actions page - impersonation action left as 'Don't apply any action'*

![Spoof / DMARC action dropdowns and safety tip options](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/55.png)
*Spoof / DMARC action dropdowns and safety tip options*

![Corrected: 'Quarantine the message' set for user impersonation](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/56.png)
*Corrected: 'Quarantine the message' set for user impersonation*

![Safety tips - 'Show user impersonation safety tip' enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/57.png)
*Safety tips - 'Show user impersonation safety tip' enabled*

![MISTAKE: Client Error - 'An error occurred when creating the policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/58.png)
*MISTAKE: Client Error - 'An error occurred when creating the policy'*

![FIX: Apply quarantine policy dropdown set to AdminOnlyAccessPolicy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/59.png)
*FIX: Apply quarantine policy dropdown set to AdminOnlyAccessPolicy*

![New anti-phishing policy created successfully after the fix](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/60.png)
*New anti-phishing policy created successfully after the fix*

![Confirmation screen - Executive Impersonation Protection is live](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/61.png)
*Confirmation screen - Executive Impersonation Protection is live*

![Anti-spam policies list - Inbound, Connection filter, Outbound defaults](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/62.png)
*Anti-spam policies list - Inbound, Connection filter, Outbound defaults*

![Create anti-spam inbound policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/63.png)
*Create anti-spam inbound policy - naming step*

![Policy named 'Strict Bulk Email Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/64.png)
*Policy named 'Strict Bulk Email Policy'*

![Users, groups and domains step for anti-spam policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/65.png)
*Users, groups and domains step for anti-spam policy*

![Tenant domain added to anti-spam policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/66.png)
*Tenant domain added to anti-spam policy scope*

![Bulk email threshold slider at default value (7)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/67.png)
*Bulk email threshold slider at default value (7)*

![Additional spam property toggles (mark as spam options)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/68.png)
*Additional spam property toggles (mark as spam options)*

![SPF / Sender ID / Test mode settings left at defaults](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/69.png)
*SPF / Sender ID / Test mode settings left at defaults*

![Bulk email threshold lowered to 5 for stricter filtering](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/70.png)
*Bulk email threshold lowered to 5 for stricter filtering*

![Actions page - Spam/Phishing quarantine actions and Zero-hour auto purge](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/71.png)
*Actions page - Spam/Phishing quarantine actions and Zero-hour auto purge*

![Allow & block list step left empty](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/72.png)
*Allow & block list step left empty*

![New anti-spam policy 'Strict Bulk Email Policy' created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/73.png)
*New anti-spam policy 'Strict Bulk Email Policy' created successfully*

![Anti-malware policies list - default policy Always on](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/74.png)
*Anti-malware policies list - default policy Always on*

![Create new anti-malware policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/75.png)
*Create new anti-malware policy - naming step*

![Users and domains step for anti-malware policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/76.png)
*Users and domains step for anti-malware policy*

![Policy named 'Custom Malware Protection Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/77.png)
*Policy named 'Custom Malware Protection Policy'*

![Protection settings - common attachments filter enabled by default](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/78.png)
*Protection settings - common attachments filter enabled by default*

!['Enable zero-hour auto purge for malware' additionally enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/79.png)
*'Enable zero-hour auto purge for malware' additionally enabled*

![New anti-malware policy created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/80.png)
*New anti-malware policy created successfully*


## Task 5 - EICAR Anti-malware Detection Test

![Official EICAR test file page (eicar.org)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/81.png)
*Official EICAR test file page (eicar.org)*

![MISTAKE: browser opened eicar.com as plain text instead of downloading it](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/82.png)
*MISTAKE: browser opened eicar.com as plain text instead of downloading it*

![FIX: Download Area - using the eicar_com.zip package instead](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/83.png)
*FIX: Download Area - using the eicar_com.zip package instead*

![eicar_com.zip (184 bytes) downloaded successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/84.png)
*eicar_com.zip (184 bytes) downloaded successfully*

![Test email composed with eicar_com.zip attached, ready to send](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/85.png)
*Test email composed with eicar_com.zip attached, ready to send*

![Quarantine list confirms detection - Quarantine reason: Malware](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/86.png)
*Quarantine list confirms detection - Quarantine reason: Malware*


## Task 6 - Safe Links Test

![Safe Links policies list - built-in Microsoft protection (On)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/87.png)
*Safe Links policies list - built-in Microsoft protection (On)*

![Create Safe Links policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/88.png)
*Create Safe Links policy - naming step*

![Policy named 'Custom Safe Links Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/89.png)
*Policy named 'Custom Safe Links Policy'*

![Users and domains step for Safe Links policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/90.png)
*Users and domains step for Safe Links policy*

![URL & click protection settings - real-time scanning enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/91.png)
*URL & click protection settings - real-time scanning enabled*

![Notification step - custom text left blank (validation risk)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/92.png)
*Notification step - custom text left blank (validation risk)*

![FIX: switched to 'Use the default notification text'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/93.png)
*FIX: switched to 'Use the default notification text'*

![New Safe Links policy created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/94.png)
*New Safe Links policy created successfully*

![Test email with a plain https://www.microsoft.com link sent to verify](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/95.png)
*Test email with a plain https://www.microsoft.com link sent to verify*


## Task 7 - Attack Simulation Training (Real Phishing Simulation)

![Attack simulation training overview page](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/96.png)
*Attack simulation training overview page*

![Select technique step - choosing Credential Harvest](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/97.png)
*Select technique step - choosing Credential Harvest*

![Full simulation wizard: technique, name, payload, target users, training, landing page, notification, launch, and review](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/98.png)
*Full simulation wizard: technique, name, payload, target users, training, landing page, notification, launch, and review*

![Results: phishing email received, fake credential page, education landing page, and admin report showing 1 click / 1 compromised](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/99.png)
*Results: phishing email received, fake credential page, education landing page, and admin report showing 1 click / 1 compromised*


## Mistakes & fixes

| # | Mistake | Fix |
|---|---------|-----|
| 1 | Mail flow rule shows "Last execution" blank in the Rules list right after it fires | Data refresh is delayed; rechecking later showed "1 hour ago" - rule had in fact run |
| 2 | Anti-phishing policy wizard: "Client Error - An error occurred when creating the policy" on Submit | Actions step: "Quarantine the message" requires "Apply quarantine policy" to be set explicitly (was left on "Select an option") - fixed by choosing AdminOnlyAccessPolicy |
| 3 | Browser opened `eicar.com` directly as plain text instead of downloading it | Used the `eicar_com.zip` package from the official download area instead, which forces a real file download |
| 4 | Quarantine message preview panel showed "Something went wrong - Primary and secondary data missing" | Cosmetic/UI-only issue; the quarantine list itself correctly showed Quarantine reason: Malware |
| 5 | Safe Links policy created with "Do not rewrite URLs, do checks via Safe Links API only" left checked by default | Test links did not visually rewrite to a safelinks.protection.outlook.com URL - this is expected API-only behaviour, not a failure; rewriting must be turned off explicitly to see wrapped URLs |
| 6 | Safe Links "Use custom notification text" selected with an empty text box | Switched to "Use the default notification text" to avoid a validation error |
| 7 | Anti-phishing policy wizard progress was lost after closing the browser mid-session | Policies are only saved on Submit; the wizard was simply restarted from Threat policies > Anti-phishing > Create |

## Key learnings

- Exchange mail flow rules are disabled by default after creation and must be manually enabled.
- Report-only / Test mode discipline (used earlier for Conditional Access policies) carries over
  naturally to mail flow rules and applies equally well here - safe to verify before enforcing.
- Anti-malware, anti-spam, and anti-phishing are three genuinely separate detection layers with
  different action pipelines (reject/quarantine/move to junk) - a full defense needs all three configured.
- The EICAR string is industry-standard and completely harmless, but the file must be delivered as
  a real file (zip) - browsers may render `.com`/`.txt` test files as plain text instead of downloading them.
- Attack Simulation Training's full lifecycle (delivery -> click -> credential harvest -> education
  landing page -> admin report) can be observed end-to-end using only the built-in test user accounts
  from earlier projects, at zero additional cost.
