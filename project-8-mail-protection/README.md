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

**Step 1: Exchange admin center home page after login**

![Exchange admin center home page after login](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/01.png)

This is the Exchange admin center (EAC) home page immediately after signing in as the tenant admin. The dashboard shows quick-access cards for mail flow, mailboxes, and reporting. This is the starting point for every Exchange-side administrative task in this project, including creating shared mailboxes, mail flow rules, and running message trace searches.

**Step 2: Mailboxes card with 'Add a shared mailbox' option**

![Mailboxes card with 'Add a shared mailbox' option](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/02.png)

The Mailboxes card on the EAC home page exposes a shortcut to create a shared mailbox directly, without needing to navigate through the full Recipients menu. 'Add a shared mailbox' was selected here to begin provisioning a mailbox that will be shared by a team rather than owned by a single user.

**Step 3: Add a shared mailbox form (Display name, Email address, Alias)**

![Add a shared mailbox form (Display name, Email address, Alias)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/03.png)

The 'Add a shared mailbox' form requires three fields: a Display name (shown to recipients), an Email address (the mailbox's login name before the @ symbol), and an Alias (an internal identifier that Exchange uses for mail routing). As a rule of thumb, the alias was set to match the email address prefix to avoid any inconsistency between the two.

**Step 4: Shared mailbox created successfully - IT Support mailbox**

![Shared mailbox created successfully - IT Support mailbox](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/04.png)

Confirmation that the shared mailbox was created successfully. Exchange notes that it may take a few minutes before the mailbox becomes fully available for adding members, since directory changes of this kind propagate asynchronously across the tenant.

**Step 5: Next step: Add users to this mailbox**

![Next step: Add users to this mailbox](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/05.png)

Immediately after creation, EAC offers a 'Next steps' panel with a direct link to 'Add users to this mailbox' — this is the delegation step where specific tenant users are granted permission to open and use the shared mailbox as if it were their own.

**Step 6: Manage shared mailbox members panel (0 items)**

![Manage shared mailbox members panel (0 items)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/06.png)

The 'Manage shared mailbox members' panel, shown here with 0 items added. This is where Full Access permission is granted — Full Access lets a delegate open the mailbox and behave as its owner, but it does not by itself allow sending mail as the shared mailbox's address (that is a separate 'Send As' permission).

**Step 7: Full list of tenant users available to add as delegates**

![Full list of tenant users available to add as delegates](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/07.png)

The full picker list of every mailbox-enabled identity in the tenant, including the five standard test accounts (Md. Imran Ahmed, Pallob Kumar Roy, Shakhawat Hossian Anik, Wardha Anjum), the admin account, and several existing distribution/shared mailboxes such as IT Support and All Company.

**Step 8: Md. Imran Ahmed and Pallob Kumar Roy selected for Full Access**

![Md. Imran Ahmed and Pallob Kumar Roy selected for Full Access](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/08.png)

Md. Imran Ahmed and Pallob Kumar Roy were selected as delegates for the shared mailbox. The admin account was deliberately left out of this list, consistent with this lab's standing practice of never using the primary admin identity as a test subject, to avoid any risk of an accidental self-lockout.

**Step 9: Shared mailbox members updated successfully**

![Shared mailbox members updated successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/09.png)

Confirmation that the shared mailbox membership update succeeded. Exchange notes that the change can take up to 60 minutes to become visible to end users in Outlook and OWA, which is a normal part of how mailbox permission changes replicate through the service.


## Task 2 - Mail Flow Rule

**Step 1: Mail flow > Rules page (empty, no rules yet)**

![Mail flow > Rules page (empty, no rules yet)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/10.png)

The Mail flow > Rules page, shown here with no rules configured yet. Mail flow rules (previously called transport rules) are the mechanism Exchange uses to apply if/then logic — such as adding recipients, blocking messages, or applying disclaimers — to every message that passes through the organization.

**Step 2: 'Add a rule' dropdown - choosing Create a new rule**

!['Add a rule' dropdown - choosing Create a new rule](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/11.png)

The 'Add a rule' dropdown lists all of the pre-built rule templates Exchange offers (message encryption, disclaimers, size filtering, restricting senders, moderation, and more). 'Create a new rule' was chosen instead, in order to build a fully custom condition/action pair from scratch.

**Step 3: New transport rule wizard - Set rule conditions step**

![New transport rule wizard - Set rule conditions step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/12.png)

The New transport rule wizard's first step, 'Set rule conditions'. This is where the rule's Name, its 'Apply this rule if' condition, its 'Do the following' action, and any 'Except if' exceptions are all defined together on a single page.

**Step 4: 'Specify words or phrases' panel - adding keyword 'confidential'**

!['Specify words or phrases' panel - adding keyword 'confidential'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/13.png)

The 'Specify words or phrases' side panel, used to define the exact keyword the rule should look for. The word 'confidential' was entered and added here — the rule will later check every outgoing message's subject line against this word list.

**Step 5: 'Do the following' dropdown showing available actions**

!['Do the following' dropdown showing available actions](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/14.png)

The 'Do the following' action dropdown, showing the categories of actions a mail flow rule can take: forwarding for approval, redirecting, blocking, adding recipients, applying a disclaimer, modifying message properties or security, and more.

**Step 6: Selecting 'to the Bcc box' as the recipient action**

![Selecting 'to the Bcc box' as the recipient action](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/15.png)

After choosing 'Add recipients', a second-level dropdown appears offering To, Cc, Bcc, or 'add the sender's manager as a recipient'. 'to the Bcc box' was selected so that a copy of any matching message is quietly sent to the admin without alerting the original recipient.

**Step 7: Select members panel - full tenant recipient list**

![Select members panel - full tenant recipient list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/16.png)

The recipient picker used to choose exactly who should receive the Bcc copy. This list mirrors the same tenant directory used elsewhere in Exchange administration.

**Step 8: Admin account selected as Bcc recipient**

![Admin account selected as Bcc recipient](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/17.png)

The admin account (rakibulhcrakib2@BOL959.onmicrosoft.com) was selected as the Bcc recipient for this rule, representing a compliance/security team member who needs visibility into potentially sensitive outgoing mail.

**Step 9: Rule condition and action confirmed before proceeding**

![Rule condition and action confirmed before proceeding](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/18.png)

With both the condition ('subject includes confidential') and the action ('Bcc the admin') now configured, the rule's logic is fully visible on the Set rule conditions page before moving on to the next wizard step.

**Step 10: Set rule settings step - Rule mode options (Enforce / Test)**

![Set rule settings step - Rule mode options (Enforce / Test)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/19.png)

The 'Set rule settings' step, where the Rule mode is chosen. Exchange offers three modes: Enforce (the rule actually takes effect), Test with Policy Tips, and Test without Policy Tips (the rule is simulated and logged, but no action is actually taken).

**Step 11: 'Test without Policy Tips' mode selected (Report-only discipline)**

!['Test without Policy Tips' mode selected (Report-only discipline)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/20.png)

'Test without Policy Tips' was selected, deliberately continuing this lab's established practice — already used for Conditional Access policies earlier in the project — of validating a new policy in a non-enforcing mode before switching it to full enforcement.

**Step 12: Review and finish - full rule summary before creation**

![Review and finish - full rule summary before creation](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/21.png)

The 'Review and finish' step lays out a complete summary of the rule's conditions, actions, and settings (mode: Audit, priority: 0, severity: not specified) so everything can be double-checked before the rule is actually created.

**Step 13: Transport rule created successfully**

![Transport rule created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/22.png)

Confirmation that the transport rule was created successfully.

**Step 14: New rule appears Disabled by default in the Rules list**

![New rule appears Disabled by default in the Rules list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/23.png)

A commonly missed detail: newly created Exchange mail flow rules are Disabled by default. The Rules list confirms this — the rule exists but will not run at all until it is manually switched on.

**Step 15: Rule manually Enabled - status panel confirms Audit mode**

![Rule manually Enabled - status panel confirms Audit mode](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/24.png)

The rule's detail panel after manually toggling it on, confirming Status: Enabled and Mode: Audit. This is the state the rule needed to be in before it could be tested with a real message.


## Task 3 - Message Trace

**Step 1: Composing test email with subject 'Confidential - Test Mail Trace'**

![Composing test email with subject 'Confidential - Test Mail Trace'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/25.png)

A test email, with the subject 'Confidential - Test Mail Trace', composed from the admin account to Md. Imran Ahmed in Outlook on the web. The subject deliberately includes the keyword 'Confidential' so that it matches the mail flow rule created in Task 2.

**Step 2: Test email visible in Sent Items**

![Test email visible in Sent Items](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/26.png)

The test email appears in Sent Items, confirming it left the mailbox successfully and is now in transit through Exchange Online, where the mail flow rule should evaluate it.

**Step 3: New message trace search form with sender/recipient filters**

![New message trace search form with sender/recipient filters](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/27.png)

The 'New message trace' search form in the Exchange admin center, used to look up the delivery history of a specific message by sender, recipient, subject, or time range. Message trace is the tool an admin or SOC analyst uses to answer 'what happened to this email?'.

**Step 4: Message trace filters confirmed (sender, recipient, time range)**

![Message trace filters confirmed (sender, recipient, time range)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/28.png)

The trace search was scoped to the exact sender (admin) and recipient (Imran) involved in the test, with the time range covering the last hour, before running the search.

**Step 5: Message trace search results - status: Delivered**

![Message trace search results - status: Delivered](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/29.png)

The message trace search results confirm the test email's final status: Delivered. This proves the message successfully reached Imran's Inbox folder.

**Step 6: Detailed trace view - 'Message events: No data available'**

![Detailed trace view - 'Message events: No data available'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/30.png)

Opening the individual trace record shows a Received -> Processed -> Delivered timeline, but the 'Message events' section reports 'No data available' at this point — the Summary report view does not surface rule-level trigger detail in real time.

**Step 7: Rules list - 'Last execution' column empty (delay observed)**

![Rules list - 'Last execution' column empty (delay observed)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/31.png)

Returning to the Rules list to check rule execution directly shows the 'Last execution' column still blank shortly after the test message was sent, suggesting execution telemetry had not yet refreshed.

**Step 8: Rules list rechecked later - 'Last execution: 1 hour ago' confirmed**

![Rules list rechecked later - 'Last execution: 1 hour ago' confirmed](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/32.png)

Rechecking the same Rules list roughly an hour later shows 'Last execution: 1 hour ago', confirming the rule genuinely had fired — the earlier blank reading was simply a reporting delay, not a failure of the rule itself.


## Task 4 - Anti-phishing, Anti-spam & Anti-malware Policies

**Step 1: Microsoft Defender - Policies & rules landing page**

![Microsoft Defender - Policies & rules landing page](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/33.png)

The Microsoft Defender portal's 'Policies & rules' landing page, the entry point for configuring every Exchange Online Protection and Defender for Office 365 policy — including anti-phishing, anti-spam, anti-malware, Safe Links, and Safe Attachments.

**Step 2: Threat policies overview - Anti-phishing, Anti-spam, Anti-malware, Safe Attachments, Safe Links**

![Threat policies overview - Anti-phishing, Anti-spam, Anti-malware, Safe Attachments, Safe Links](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/34.png)

The Threat policies overview lists every protection category side by side: Anti-phishing, Anti-spam, Anti-malware, Safe Attachments, and Safe Links, plus supporting rules such as Tenant Allow/Block Lists and Quarantine policies.

**Step 3: Anti-phishing policies list - default Office365 AntiPhish policy**

![Anti-phishing policies list - default Office365 AntiPhish policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/35.png)

The Anti-phishing policies list, showing the built-in 'Office365 AntiPhish Default' policy that applies to every user in the tenant at the lowest priority. A higher-priority custom policy can be layered on top of this default for more targeted protection.

**Step 4: Create new anti-phishing policy - naming step**

![Create new anti-phishing policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/36.png)

The first step of the custom anti-phishing policy wizard: naming the new policy 'Executive Impersonation Protection' and giving it a description explaining its purpose.

**Step 5: Users, groups and domains step - adding tenant domain**

![Users, groups and domains step - adding tenant domain](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/37.png)

The 'Users, groups, and domains' step, where the scope of the new policy is defined. A domain was entered here so the policy applies tenant-wide rather than to a single user or group.

**Step 6: Domain BOL959.onmicrosoft.com added to policy scope**

![Domain BOL959.onmicrosoft.com added to policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/38.png)

The tenant domain BOL959.onmicrosoft.com is confirmed as added to the policy's scope.

**Step 7: Phishing threshold & protection - Impersonation section**

![Phishing threshold & protection - Impersonation section](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/39.png)

The 'Phishing threshold & protection' step is where the real protection logic lives: the Impersonation section lets specific high-value users (like executives or admins) be individually protected from having their name or domain spoofed by an attacker.

**Step 8: 'Enable users to protect' checkbox not yet applied (first attempt)**

!['Enable users to protect' checkbox not yet applied (first attempt)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/40.png)

An initial attempt to enable the 'Enable users to protect' checkbox did not register — the box is still shown unchecked here, which prompted a retry by clicking directly on the checkbox itself rather than the surrounding text.

**Step 9: 'Enable users to protect' successfully checked**

!['Enable users to protect' successfully checked](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/41.png)

The checkbox is now confirmed checked, enabling per-user impersonation protection for up to 350 internal and external senders.

**Step 10: Manage senders for impersonation protection panel**

![Manage senders for impersonation protection panel](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/42.png)

The 'Manage senders for impersonation protection' panel, where specific individuals are added to the protected list. Adding someone here means Defender will flag any message that appears to impersonate their name or email address.

**Step 11: Add user dialog - admin account added as protected sender**

![Add user dialog - admin account added as protected sender](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/43.png)

The 'Add user' dialog with the admin account (Rakibul Hoque Chowdhury) added to the protected senders list, representing the account most likely to be targeted by a real-world impersonation attack.

**Step 12: Protected sender confirmed in the Add user list**

![Protected sender confirmed in the Add user list](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/44.png)

The admin account is confirmed present in the Add user list, ready to be saved back to the main policy.

**Step 13: Session resumed - Anti-phishing list shows only default policy (draft lost)**

![Session resumed - Anti-phishing list shows only default policy (draft lost)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/45.png)

After a browser restart, the Anti-phishing policies list shows only the original default policy — the in-progress custom policy from the earlier session was never actually saved, because Exchange and Defender wizards only persist configuration once the final Submit step is completed.

**Step 14: Restarting wizard - Policy name step**

![Restarting wizard - Policy name step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/46.png)

The wizard was restarted from scratch at the Policy name step, since the previous session's progress had been lost.

**Step 15: Policy named 'Executive Impersonation Protection'**

![Policy named 'Executive Impersonation Protection'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/47.png)

The policy was named 'Executive Impersonation Protection' again, along with a matching description, and the wizard proceeded normally this time.

**Step 16: Users, groups and domains step (second attempt)**

![Users, groups and domains step (second attempt)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/48.png)

The 'Users, groups, and domains' step, revisited on the second attempt.

**Step 17: Tenant domain re-added to policy scope**

![Tenant domain re-added to policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/49.png)

The tenant domain was re-added to the policy's scope on this second attempt.

**Step 18: Phishing threshold & protection page reloaded**

![Phishing threshold & protection page reloaded](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/50.png)

The Phishing threshold & protection page reloaded cleanly on the second attempt, with the threshold left at its default 'Standard' value.

**Step 19: Enable users to protect checked, Manage senders link visible**

![Enable users to protect checked, Manage senders link visible](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/51.png)

'Enable users to protect' was checked successfully this time, exposing the 'Manage 0 sender(s)' link needed to add the protected user.

**Step 20: Manage senders panel - admin already listed as protected sender**

![Manage senders panel - admin already listed as protected sender](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/52.png)

The Manage senders panel already shows the admin account listed as a protected sender — confirming the entry carried over correctly this time.

**Step 21: Sender confirmed before clicking Done**

![Sender confirmed before clicking Done](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/53.png)

The sender entry was confirmed once more immediately before clicking Done to close the panel.

**Step 22: Actions page - impersonation action left as 'Don't apply any action'**

![Actions page - impersonation action left as 'Don't apply any action'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/54.png)

The Actions step, where the response to a detected impersonation attempt is configured. At this point the 'If a message is detected as user impersonation' action was still left at its default, unhelpful value of 'Don't apply any action'.

**Step 23: Spoof / DMARC action dropdowns and safety tip options**

![Spoof / DMARC action dropdowns and safety tip options](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/55.png)

Further down the same Actions page: the Spoof and DMARC-related action dropdowns (largely left at their sensible Microsoft-recommended defaults) and the Safety tips & indicators checkboxes that control what warning banners end users see on suspicious mail.

**Step 24: Corrected: 'Quarantine the message' set for user impersonation**

![Corrected: 'Quarantine the message' set for user impersonation](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/56.png)

The critical fix: the 'If a message is detected as user impersonation' action was changed from 'Don't apply any action' to 'Quarantine the message', so that a genuine impersonation attempt would actually be intercepted rather than silently delivered.

**Step 25: Safety tips - 'Show user impersonation safety tip' enabled**

![Safety tips - 'Show user impersonation safety tip' enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/57.png)

'Show user impersonation safety tip' was additionally enabled, so that end users see a visible warning banner directly inside a suspicious message.

**Step 26: MISTAKE: Client Error - 'An error occurred when creating the policy'**

![MISTAKE: Client Error - 'An error occurred when creating the policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/58.png)

MISTAKE: submitting the policy at this point produced a 'Client Error — An error occurred when creating the policy' dialog, with no further explanation from the UI about what specifically had failed validation.

**Step 27: FIX: Apply quarantine policy dropdown set to AdminOnlyAccessPolicy**

![FIX: Apply quarantine policy dropdown set to AdminOnlyAccessPolicy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/59.png)

THE FIX: going back into the Actions step revealed that choosing 'Quarantine the message' requires an explicit 'Apply quarantine policy' selection — it had been left on the placeholder 'Select an option'. Setting it to 'AdminOnlyAccessPolicy' resolved the hidden validation requirement.

**Step 28: New anti-phishing policy created successfully after the fix**

![New anti-phishing policy created successfully after the fix](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/60.png)

With the quarantine policy explicitly selected, the new anti-phishing policy 'Executive Impersonation Protection' was created successfully on the next Submit attempt.

**Step 29: Confirmation screen - Executive Impersonation Protection is live**

![Confirmation screen - Executive Impersonation Protection is live](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/61.png)

The completed policy's confirmation screen, with links to view the policy directly and to review all anti-phishing policies in the tenant.

**Step 30: Anti-spam policies list - Inbound, Connection filter, Outbound defaults**

![Anti-spam policies list - Inbound, Connection filter, Outbound defaults](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/62.png)

The Anti-spam policies list shows three separate built-in policies that ship with every tenant: an Inbound policy, a Connection filter policy, and an Outbound policy — each covering a different stage of the spam-filtering pipeline.

**Step 31: Create anti-spam inbound policy - naming step**

![Create anti-spam inbound policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/63.png)

The first step of the custom Inbound anti-spam policy wizard, which follows six stages: naming, scope, bulk-email threshold and spam properties, actions, allow/block list, and review.

**Step 32: Policy named 'Strict Bulk Email Policy'**

![Policy named 'Strict Bulk Email Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/64.png)

The policy was named 'Strict Bulk Email Policy', with a description explaining its intent to apply a stricter bulk-mail threshold than the tenant default.

**Step 33: Users, groups and domains step for anti-spam policy**

![Users, groups and domains step for anti-spam policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/65.png)

The Users, groups, and domains step for this anti-spam policy, used to scope it to the whole tenant domain.

**Step 34: Tenant domain added to anti-spam policy scope**

![Tenant domain added to anti-spam policy scope](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/66.png)

The tenant domain BOL959.onmicrosoft.com confirmed as added to the anti-spam policy's scope.

**Step 35: Bulk email threshold slider at default value (7)**

![Bulk email threshold slider at default value (7)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/67.png)

The 'Bulk email threshold & spam properties' step, shown with the slider at its Microsoft default value of 7 — a higher number means more bulk/marketing-style email is allowed through rather than flagged.

**Step 36: Additional spam property toggles (mark as spam options)**

![Additional spam property toggles (mark as spam options)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/68.png)

Further down the same page, the additional 'Mark as spam' toggles for suspicious message characteristics (empty messages, embedded scripts, form tags, and similar HTML-based signals) were reviewed and left at their default Off state, since enabling all of them would be overly aggressive for a small test tenant.

**Step 37: SPF / Sender ID / Test mode settings left at defaults**

![SPF / Sender ID / Test mode settings left at defaults](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/69.png)

SPF hard-fail, Sender ID hard-fail, backscatter detection, language/country filtering, and Test mode settings were all reviewed and left at their defaults, since none of these advanced signals were needed for this specific demonstration.

**Step 38: Bulk email threshold lowered to 5 for stricter filtering**

![Bulk email threshold lowered to 5 for stricter filtering](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/70.png)

The bulk email threshold slider was deliberately lowered from the default 7 down to 5, making the policy noticeably stricter — a smaller number means Exchange treats less obviously 'bulk-like' mail as junk.

**Step 39: Actions page - Spam/Phishing quarantine actions and Zero-hour auto purge**

![Actions page - Spam/Phishing quarantine actions and Zero-hour auto purge](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/71.png)

The Actions step confirms the response pipeline for this policy: spam and high-confidence spam are moved to the Junk Email folder, phishing and high-confidence phishing are quarantined against DefaultFullAccessPolicy and AdminOnlyAccessPolicy respectively, and Zero-hour auto purge (ZAP) is enabled for both phishing and spam.

**Step 40: Allow & block list step left empty**

![Allow & block list step left empty](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/72.png)

The Allow & block list step was left empty, since no specific senders or domains needed to be pre-approved or pre-blocked for this test policy.

**Step 41: New anti-spam policy 'Strict Bulk Email Policy' created successfully**

![New anti-spam policy 'Strict Bulk Email Policy' created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/73.png)

Confirmation that the 'Strict Bulk Email Policy' anti-spam policy was created successfully, with no errors this time.

**Step 42: Anti-malware policies list - default policy Always on**

![Anti-malware policies list - default policy Always on](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/74.png)

The Anti-malware policies list, showing the single built-in Default policy that is always on for the entire tenant.

**Step 43: Create new anti-malware policy - naming step**

![Create new anti-malware policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/75.png)

The first step of the custom anti-malware policy wizard — naming and describing the new policy, which follows a shorter four-step flow than the anti-phishing and anti-spam wizards.

**Step 44: Users and domains step for anti-malware policy**

![Users and domains step for anti-malware policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/76.png)

The Users and domains step for the anti-malware policy, with the tenant domain added to scope it organization-wide.

**Step 45: Policy named 'Custom Malware Protection Policy'**

![Policy named 'Custom Malware Protection Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/77.png)

The policy was named 'Custom Malware Protection Policy', with a description noting its focus on enhanced attachment filtering.

**Step 46: Protection settings - common attachments filter enabled by default**

![Protection settings - common attachments filter enabled by default](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/78.png)

The Protection settings step, showing the 'Enable the common attachments filter' option already checked by default — this blocks over 40 well-known dangerous file extensions (.exe, .apk, .bat, and similar) automatically, rejecting the message with a non-delivery receipt when one is found.

**Step 47: 'Enable zero-hour auto purge for malware' additionally enabled**

!['Enable zero-hour auto purge for malware' additionally enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/79.png)

'Enable zero-hour auto purge for malware' was additionally switched on, following Microsoft's own recommendation — this lets Exchange retroactively remove a message from an inbox if a malware signature is discovered after the message was already delivered.

**Step 48: New anti-malware policy created successfully**

![New anti-malware policy created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/80.png)

Confirmation that the 'Custom Malware Protection Policy' anti-malware policy was created successfully.


## Task 5 - EICAR Anti-malware Detection Test

**Step 1: Official EICAR test file page (eicar.org)**

![Official EICAR test file page (eicar.org)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/81.png)

The official EICAR test file page (eicar.org). EICAR is an industry-standard, completely harmless test string that every major antivirus and anti-malware engine is configured to recognize and flag as if it were a real virus — it exists specifically so that detection systems can be verified safely, without using real malware.

**Step 2: MISTAKE: browser opened eicar.com as plain text instead of downloading it**

![MISTAKE: browser opened eicar.com as plain text instead of downloading it](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/82.png)

MISTAKE: clicking the plain 'eicar.com' download link caused the browser to open the file's raw content directly as text in a new tab, instead of saving it to disk as an actual file that could be attached to an email.

**Step 3: FIX: Download Area - using the eicar_com.zip package instead**

![FIX: Download Area - using the eicar_com.zip package instead](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/83.png)

THE FIX: the official Download Area offers four packaging options (eicar.com, eicar.com.txt, eicar.com.zip, eicar.com-2.zip). The zipped version was used instead, since a browser cannot render a zip archive inline the way it can render a plain text file.

**Step 4: eicar_com.zip (184 bytes) downloaded successfully**

![eicar_com.zip (184 bytes) downloaded successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/84.png)

The eicar_com.zip file (184 bytes) downloaded successfully to disk this time, confirmed by the browser's download notification showing 'Done'.

**Step 5: Test email composed with eicar_com.zip attached, ready to send**

![Test email composed with eicar_com.zip attached, ready to send](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/85.png)

A test email was composed in Outlook on the web with eicar_com.zip attached, subject 'EICAR Anti-Malware Test', and a body clarifying that this is a safe test file rather than a real virus — sent to Md. Imran Ahmed to trigger the anti-malware policy's scanning.

**Step 6: Quarantine list confirms detection - Quarantine reason: Malware**

![Quarantine list confirms detection - Quarantine reason: Malware](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/86.png)

The Quarantine list confirms the anti-malware policy worked exactly as intended: the message (subject 'Undeliverable: EICA...') was automatically caught and quarantined, with Quarantine reason listed as Malware — direct proof that the 'Custom Malware Protection Policy' built earlier in Task 4 is functioning correctly.


## Task 6 - Safe Links Test

**Step 1: Safe Links policies list - built-in Microsoft protection (On)**

![Safe Links policies list - built-in Microsoft protection (On)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/87.png)

The Safe Links policies list, showing Microsoft's built-in protection policy already turned On for the tenant at the lowest priority.

**Step 2: Create Safe Links policy - naming step**

![Create Safe Links policy - naming step](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/88.png)

The first step of the custom Safe Links policy wizard — naming and describing the new policy.

**Step 3: Policy named 'Custom Safe Links Policy'**

![Policy named 'Custom Safe Links Policy'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/89.png)

The policy was named 'Custom Safe Links Policy', with a description noting its purpose of URL scanning and time-of-click protection for all users.

**Step 4: Users and domains step for Safe Links policy**

![Users and domains step for Safe Links policy](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/90.png)

The Users and domains step for the Safe Links policy, scoping it to the entire tenant domain.

**Step 5: URL & click protection settings - real-time scanning enabled**

![URL & click protection settings - real-time scanning enabled](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/91.png)

The 'URL & click protection settings' step, with real-time URL scanning enabled for Email, Teams, and Office 365 Apps, and the 'Do not rewrite URLs, do checks via Safe Links API only' option left checked by default — meaning links are verified in the background rather than being visually rewritten.

**Step 6: Notification step - custom text left blank (validation risk)**

![Notification step - custom text left blank (validation risk)](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/92.png)

The Notification step defaulted to 'Use custom notification text' with an empty text box, which risked failing validation on submission since no custom text had actually been written.

**Step 7: FIX: switched to 'Use the default notification text'**

![FIX: switched to 'Use the default notification text'](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/93.png)

THE FIX: the notification option was switched to 'Use the default notification text' instead, avoiding the empty-field validation risk entirely by relying on Microsoft's standard wording.

**Step 8: New Safe Links policy created successfully**

![New Safe Links policy created successfully](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/94.png)

Confirmation that the 'Custom Safe Links Policy' was created successfully and is now in effect immediately.

**Step 9: Test email with a plain https://www.microsoft.com link sent to verify**

![Test email with a plain https://www.microsoft.com link sent to verify](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/95.png)

A test email containing a plain https://www.microsoft.com link was sent to verify Safe Links behaviour. Because the policy uses API-only checking rather than URL rewriting, the link displays in its original, unrewritten form — this is expected behaviour for this configuration, not a sign that scanning failed.


## Task 7 - Attack Simulation Training (Real Phishing Simulation)

**Step 1: Attack simulation training overview page**

![Attack simulation training overview page](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/96.png)

The Attack simulation training overview page in Microsoft Defender, offering a choice between launching an instant pre-built simulation or building a fully custom one from scratch.

**Step 2: Select technique step - choosing Credential Harvest**

![Select technique step - choosing Credential Harvest](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/97.png)

The 'Select technique' step of the simulation wizard, listing the MITRE ATT&CK-based social engineering techniques available: Malware Attachment, Link in Attachment, Link to Malware, Drive-by URL, OAuth Consent Grant, and How-to Guide. Credential Harvest — the classic fake-login-page technique — was chosen as the most realistic and widely used real-world phishing pattern.

**Step 3: Full simulation wizard: technique, name, payload, target users, training, landing page, notification, launch, and review**

![Full simulation wizard: technique, name, payload, target users, training, landing page, notification, launch, and review](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/98.png)

The full simulation build-out, captured across the remaining wizard steps: the simulation was named 'Phishing Awareness Test 1', an invoice-themed payload from Microsoft's Global payload library was selected, all four standard test accounts (Md. Imran Ahmed, Pallob Kumar Roy, Shakhawat Hossian Anik, Wardha Anjum) were targeted, Microsoft's recommended training was assigned automatically based on user results, a Microsoft landing page template was applied for the post-click education page, the default end-user notification was kept, and the simulation was configured to launch immediately and run for two days before the final Review screen confirmed every setting ahead of Submit.

**Step 4: Results: phishing email received, fake credential page, education landing page, and admin report showing 1 click / 1 compromised**

![Results: phishing email received, fake credential page, education landing page, and admin report showing 1 click / 1 compromised](https://raw.githubusercontent.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/main/project-8-mail-protection/images/99.png)

The results, captured directly from Md. Imran Ahmed's inbox and the admin reporting view: the simulated phishing email ('Departamento de Finanzas', spoofed domain bankmenia.fr) landed in the inbox, clicking through led to a realistic fake Microsoft credential page, submitting a password there redirected to a friendly, non-punitive education page explaining that this was a security team exercise, and the admin's Attack simulation report confirmed the full outcome: 1 click and 1 compromised credential recorded for that user, closing the loop on the entire simulation lifecycle from delivery through to reporting.


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
