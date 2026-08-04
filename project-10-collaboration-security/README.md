# Project 10 — Collaboration & Cloud App Security

**Goal:** stand up a standalone SharePoint site and test its external-sharing policy, schedule a Teams meeting with an external guest to compare against SharePoint's sharing behavior, explore Microsoft Defender for Cloud Apps end-to-end (Activity log, App Connectors, Governance), and build a custom File Policy that flags externally-shared files — all on free-tier M365 Business Premium + EMS E5 trial licensing.

**Certification focus:** M365 / SC-200

---

## Concept

Once a file or a meeting leaves an organization's boundary, three separate layers have to work
together for that action to be both usable and safe:

1. **Policy** — the rules set in advance (site-level SharePoint sharing settings, Teams external
   access policies) that define what's even allowed to happen.
2. **Enforcement** — what actually happens the moment a person shares a file or invites someone
   external. This is where policy gets tested against reality.
3. **Visibility** — the monitoring layer (Defender for Cloud Apps' Activity log, File Policies)
   that watches for these events after the fact and can act on risky ones automatically.

This project deliberately exercised all three layers back to back, on the same test data,
so that a break in any one layer would show up clearly against the other two. That's exactly
what happened: the enforcement layer for SharePoint sharing silently failed, while the same
test on Teams (a different app, same policy tier) succeeded — isolating the fault to one
specific pipeline instead of leaving it as a vague "sharing didn't work" result.

---

## Why this is a long post

Most collaboration-security walkthroughs show a clean success path: turn sharing on, share a file, done. This one shows a SharePoint share invite that displayed a success confirmation and then silently failed to persist, a four-step root-cause chase through Entra External Identities that came up clean, a Teams meeting invite to the exact same external address that worked and proved the failure was SharePoint-specific, a brand-new Defender for Cloud Apps connector that failed with an HTTP 400 the moment it was turned on, and a trail through three different Microsoft Purview surfaces (including one that's been retired since November 2024) before finding the underlying audit-pipeline issue. None of it was hidden — it's the actual sequence, screenshot by screenshot.

---

## What was built

| # | Task | Outcome |
|---|------|---------|
| 10.1 | SharePoint site + external sharing | Standalone Communication site (`ExternalShare-Test`), site-level sharing set to *New and existing guests*, external share invite attempted |
| 10.2 | Teams meeting + external access | Dedicated Team (`External-Meeting-Test`), Teams meeting scheduled with an external Gmail attendee |
| 10.3 | Defender for Cloud Apps | Activity log, App Connectors, and Governance explored; Microsoft 365 connector provisioned |
| 10.4 | File Policy | Custom DLP-style policy (`Flag-Externally-Shared-Files`) created against the Access level = External/Public condition |

---

## Table of contents

- [10.1 — SharePoint Site Collection & External Sharing](#sharepoint-site-collection-external-sharing)
- [10.2 — Teams Meetings & External Access](#teams-meetings-external-access)
- [10.3 — Exploring Defender for Cloud Apps](#exploring-defender-for-cloud-apps)
- [10.4 — File Policy](#file-policy)
- [Mistakes & Fixes](#mistakes--fixes)
- [Key Learnings](#key-learnings)

---

## 10.1 — SharePoint Site Collection & External Sharing

![SharePoint Admin Center — Active sites before creating the new test site.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/01.png)
*SharePoint Admin Center — Active sites before creating the new test site.*

![Create a site — choosing between Team site and Communication site.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/02.png)
*Create a site — choosing between Team site and Communication site.*

![Selecting the Standard Communication template.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/03.png)
*Selecting the Standard Communication template.*

![Template preview before confirming.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/04.png)
*Template preview before confirming.*

![Naming the site ExternalShare-Test and confirming the URL.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/05.png)
*Naming the site ExternalShare-Test and confirming the URL.*

![Setting the site owner.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/06.png)
*Setting the site owner.*

![Language and timezone confirmation before creation.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/07.png)
*Language and timezone confirmation before creation.*

![ExternalShare-Test now live in Active sites.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/08.png)
*ExternalShare-Test now live in Active sites.*

![Site details flyout panel — General tab.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/09.png)
*Site details flyout panel — General tab.*

![Settings tab — the four external file sharing levels (Anyone / New and existing guests / Existing guests / Only people in your organization).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/10.png)
*Settings tab — the four external file sharing levels (Anyone / New and existing guests / Existing guests / Only people in your organization).*

![External file sharing changed to 'New and existing guests' and saved.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/11.png)
*External file sharing changed to 'New and existing guests' and saved.*

![The live ExternalShare-Test site home page.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/12.png)
*The live ExternalShare-Test site home page.*

![Empty Documents library before upload.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/13.png)
*Empty Documents library before upload.*

![Test file (Study Abroad.xlsx) uploaded successfully.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/14.png)
*Test file (Study Abroad.xlsx) uploaded successfully.*

![File selected, Share command available in the toolbar.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/15.png)
*File selected, Share command available in the toolbar.*

![Share dialog opened for the test file.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/16.png)
*Share dialog opened for the test file.*

![Permission level options — Can edit / Can view / Can't download.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/17.png)
*Permission level options — Can edit / Can view / Can't download.*

![Link settings — 'People in BOL' selected by default; note there is no bare 'Anyone' option because the site-level policy only allows guests, not full public sharing.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/18.png)
*Link settings — 'People in BOL' selected by default; note there is no bare 'Anyone' option because the site-level policy only allows guests, not full public sharing.*

![Link audience switched to 'People you choose' to enable external sharing.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/19.png)
*Link audience switched to 'People you choose' to enable external sharing.*

![Back at the Share dialog, ready to add an external recipient.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/20.png)
*Back at the Share dialog, ready to add an external recipient.*

![Typing a partial address returns only internal directory suggestions.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/21.png)
*Typing a partial address returns only internal directory suggestions.*

![A full external Gmail address is correctly recognized as an external contact.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/22.png)
*A full external Gmail address is correctly recognized as an external contact.*

![Confirmation message: invite sent to the external address.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/23.png)
*Confirmation message: invite sent to the external address.*

![Mistake found — Manage Access shows 'This file has not been shared with anyone yet,' despite the success confirmation moments earlier.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/24.png)
*Mistake found — Manage Access shows 'This file has not been shared with anyone yet,' despite the success confirmation moments earlier.*

![Root-cause check #1 — Entra External Identities guest invite restrictions, set to the most permissive option.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/25.png)
*Root-cause check #1 — Entra External Identities guest invite restrictions, set to the most permissive option.*

![Root-cause check #2 — collaboration restrictions allow invitations to any domain.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/26.png)
*Root-cause check #2 — collaboration restrictions allow invitations to any domain.*

![Root-cause check #3 — Email one-time-passcode identity provider is already configured.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/27.png)
*Root-cause check #3 — Email one-time-passcode identity provider is already configured.*

---

## 10.2 — Teams Meetings & External Access

![Teams Admin Center — External access policies summary (4 default policies, 0 custom).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/28.png)
*Teams Admin Center — External access policies summary (4 default policies, 0 custom).*

![Global (Org-wide default) policy — domain management set to 'Use organization settings.'](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/29.png)
*Global (Org-wide default) policy — domain management set to 'Use organization settings.'*

![Global policy communication toggles — chat/meetings with unmanaged Microsoft accounts is On.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/30.png)
*Global policy communication toggles — chat/meetings with unmanaged Microsoft accounts is On.*

![Teams client — existing teams (BOL, Managing-Team) before creating a new one.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/31.png)
*Teams client — existing teams (BOL, Managing-Team) before creating a new one.*

![Your teams and channels — list view with the Create team entry point.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/32.png)
*Your teams and channels — list view with the Create team entry point.*

![Create a team — from scratch, empty form.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/33.png)
*Create a team — from scratch, empty form.*

![Create a team — External-Meeting-Test filled in, Private, General channel.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/34.png)
*Create a team — External-Meeting-Test filled in, Private, General channel.*

![Add members step — this dialog itself supports inviting external guests directly.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/35.png)
*Add members step — this dialog itself supports inviting external guests directly.*

![External-Meeting-Test team created and visible in the left navigation.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/36.png)
*External-Meeting-Test team created and visible in the left navigation.*

![Teams Calendar view before scheduling the test meeting.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/37.png)
*Teams Calendar view before scheduling the test meeting.*

![New event form with the Teams meeting toggle.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/38.png)
*New event form with the Teams meeting toggle.*

![Meeting filled in — the external Gmail attendee shows status 'Unknown,' confirming Teams correctly recognized it as an external, unmanaged account.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/39.png)
*Meeting filled in — the external Gmail attendee shows status 'Unknown,' confirming Teams correctly recognized it as an external, unmanaged account.*

![After Send — two overlapping calendar entries appeared briefly (later resolved to one).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/40.png)
*After Send — two overlapping calendar entries appeared briefly (later resolved to one).*

![Contrast finding — the meeting invite email actually arrived in the external Gmail inbox with full RSVP controls, unlike the SharePoint share invite.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/41.png)
*Contrast finding — the meeting invite email actually arrived in the external Gmail inbox with full RSVP controls, unlike the SharePoint share invite.*

![The scheduled meeting appears correctly in Teams Calendar with a working Join button.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/42.png)
*The scheduled meeting appears correctly in Teams Calendar with a working Join button.*

---

## 10.3 — Exploring Defender for Cloud Apps

![Microsoft Defender home page with the Cloud apps navigation expanded (Cloud discovery, Cloud app catalog, OAuth apps, Activity log, Governance log, Policies).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/43.png)
*Microsoft Defender home page with the Cloud apps navigation expanded (Cloud discovery, Cloud app catalog, OAuth apps, Activity log, Governance log, Policies).*

![Activity log returns 'No activities found' for the default filters.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/44.png)
*Activity log returns 'No activities found' for the default filters.*

![App Connectors page shows 0 connected apps — the root cause of the empty activity log.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/45.png)
*App Connectors page shows 0 connected apps — the root cause of the empty activity log.*

![Connect an app — list of available connectors.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/46.png)
*Connect an app — list of available connectors.*

![Selecting Microsoft 365 components to connect, including Microsoft 365 activities.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/47.png)
*Selecting Microsoft 365 components to connect, including Microsoft 365 activities.*

!['Great, Microsoft 365 is connected' confirmation.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/48.png)
*'Great, Microsoft 365 is connected' confirmation.*

![The new connector shows a Connection error status almost immediately.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/49.png)
*The new connector shows a Connection error status almost immediately.*

![Connection error detail — Get Users succeeded, but Get Events failed with HTTP 400 Bad Request.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/50.png)
*Connection error detail — Get Users succeeded, but Get Events failed with HTTP 400 Bad Request.*

![Root-cause trail #1 — the new Microsoft Purview governance portal fails to list accounts (first-party app service principal not present).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/51.png)
*Root-cause trail #1 — the new Microsoft Purview governance portal fails to list accounts (first-party app service principal not present).*

![Root-cause trail #2 — the classic Microsoft Purview compliance portal has been retired as of November 2024.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/52.png)
*Root-cause trail #2 — the classic Microsoft Purview compliance portal has been retired as of November 2024.*

![Root-cause trail #3 — the current Purview Audit search page, before running a search.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/53.png)
*Root-cause trail #3 — the current Purview Audit search page, before running a search.*

![Root-cause trail #4 — Audit search itself fails with 'Failed to load data,' tying the whole chain together.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/54.png)
*Root-cause trail #4 — Audit search itself fails with 'Failed to load data,' tying the whole chain together.*

---

## 10.4 — File Policy

![Cloud Apps Policies page — 25 built-in policies already present, plus a retirement notice for File policies (Jan 6, 2027).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/55.png)
*Cloud Apps Policies page — 25 built-in policies already present, plus a retirement notice for File policies (Jan 6, 2027).*

![Create file policy — base fields (template, name, severity, category, description).](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/56.png)
*Create file policy — base fields (template, name, severity, category, description).*

![File-matching filter — Access level equals Public (Internet), External, Public; scoped to all files and all file owners.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/57.png)
*File-matching filter — Access level equals Public (Internet), External, Public; scoped to all files and all file owners.*

![Alerts and Governance actions section, with OneDrive and SharePoint Online available as targets.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/58.png)
*Alerts and Governance actions section, with OneDrive and SharePoint Online available as targets.*

![SharePoint Online governance actions expanded — 'Send policy-match digest to file owner' selected; 'Put in admin quarantine' and 'Apply sensitivity label' are greyed out, needing separate prerequisite setup.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/59.png)
*SharePoint Online governance actions expanded — 'Send policy-match digest to file owner' selected; 'Put in admin quarantine' and 'Apply sensitivity label' are greyed out, needing separate prerequisite setup.*

![Final policy configuration — name, severity, description, and the access-level filter confirmed.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/60.png)
*Final policy configuration — name, severity, description, and the access-level filter confirmed.*

![Alert-per-match enabled, governance action confirmed, ready to create.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/61.png)
*Alert-per-match enabled, governance action confirmed, ready to create.*

![Flag-Externally-Shared-Files policy created and Active in the policy list.](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-10-collaboration-security/images/62.png)
*Flag-Externally-Shared-Files policy created and Active in the policy list.*

---

## Mistakes & Fixes

| # | What happened | Root cause / fix |
|---|----------------|-------------------|
| 1 | SharePoint file-share invite showed a success confirmation, but *Manage Access* later reported "This file has not been shared with anyone yet" | Checked Entra External Identities guest invite restrictions (unrestricted), domain collaboration restrictions (unrestricted), and the Email OTP identity provider (already configured) — all clean. Root cause not fully isolated; documented as an open finding rather than papered over. |
| 2 | Same external Gmail address never received the SharePoint share email | Consistent with #1 — the invite never persisted server-side, so no email was ever queued. |
| 3 | Two overlapping "External Access Test Meeting" entries briefly appeared on the Teams calendar after sending the invite | Resolved on refresh to a single event; likely a transient calendar-sync render, not a duplicate send — the Gmail inbox only ever received one invite. |
| 4 | Contrast finding: the Teams meeting invite to the same external Gmail address *did* arrive, with full RSVP controls | Confirms the SharePoint failure (#1) was specific to SharePoint's guest-invite pipeline, not a tenant-wide email or Entra External Identities problem. |
| 5 | Defender for Cloud Apps Activity log returned "No activities found" | The Office 365 App Connector had never been provisioned. The XDR data connectors verified back in Project 1 are alert-level integrations, not activity-level ingestion — they're separate systems. |
| 6 | Immediately after connecting the Microsoft 365 App Connector, it showed a Connection error | Detail view showed `Get Users: Success` but `Get Events: HttpRequestFailureWithBody: status code: 400`. Traced through the new Purview governance portal (failed — first-party app service principal not present in tenant), the classic Purview compliance portal (retired since November 2024), to the current Purview Audit search page, which itself failed with "Failed to load data." Root cause points to the tenant's Unified Audit Logging pipeline, not a Cloud Apps misconfiguration. Documented as an open finding for future follow-up. |
| 7 | File Policy `Flag-Externally-Shared-Files` matched 0 files immediately after creation | Consistent with #1 and #6 — the test file's share never actually persisted as "External," and the same audit-pipeline gap likely delays or blocks policy-match scanning. |
| 8 | "Put in admin quarantine" and "Apply sensitivity label" governance actions were greyed out when building the File Policy | These require separate prerequisite setup (an admin quarantine location and a defined sensitivity-label taxonomy) that hasn't been configured yet in this lab. |

---

## Key learnings

1. **A "success" confirmation in the UI is not proof of a persisted state.** SharePoint's share dialog showed a clean success message for an invite that never actually took effect — always verify through a second, independent view (like Manage Access) before trusting the first confirmation.
2. **Alert-level integration and activity-level ingestion are different systems.** The Defender XDR connectors verified as "Connected" back in Project 1 say nothing about whether Defender for Cloud Apps' Activity log or App Connectors are populated — those need their own, separate provisioning step.
3. **Site-level SharePoint sharing policy constrains what's even offered at the file level.** Setting the site to "New and existing guests" (not "Anyone") meant the file-share dialog never even presented a bare public-link option — the ceiling set at the site enforces itself automatically downstream.
4. **Comparing two features against the same test input isolates the fault.** Sending an invite to the identical external email through SharePoint (failed) and Teams (succeeded) in the same session proved the problem was SharePoint-specific rather than a tenant-wide guest/email issue — a technique worth repeating for future troubleshooting.
5. **Microsoft's compliance/audit surface is mid-migration.** In a single troubleshooting chain this project hit a brand-new Purview governance portal with a missing service principal, a fully retired classic compliance portal (November 2024), and a current Audit search page that itself failed to load — a reminder to expect surface churn in trial tenants and to verify at each hop rather than assuming continuity.
6. **Some governance actions have silent prerequisites.** File Policy options like "Put in admin quarantine" and "Apply sensitivity label" are greyed out until their supporting infrastructure (quarantine location, label taxonomy) exists — the UI disables rather than errors, so it's easy to miss why an option isn't available.
7. **Report-only discipline extends naturally to Cloud App governance actions.** Choosing the non-destructive "Send policy-match digest to file owner" action over "Remove external users" or "Trash" for a first-pass policy keeps this consistent with the Audit/Report-only-first approach used for Conditional Access and mail flow rules in earlier projects.