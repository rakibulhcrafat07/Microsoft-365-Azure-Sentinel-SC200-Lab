# Project 2 — Tenant, Users & Governance Foundation

**Part of: [Zero-to-SOC](./README.md) — a self-funded, multi-certification cloud security lab**

This document is the full, unabridged record of Project 2 — every sub-task, in order, with the screenshot that proves it happened. It's longer than a highlight reel on purpose: the goal is a complete reference to look back on, not a marketing summary.

**Environment:** `BOL959.onmicrosoft.com` tenant · Microsoft 365 Business Premium + Entra ID P2 · Azure ($200 free credit)

---

## Contents

- [2.1 Organization profile](#21-organization-profile)
- [2.2 User provisioning — User 1 (Md. Imran Ahmed)](#22-user-provisioning--user-1-md-imran-ahmed)
- [2.3 User provisioning — User 2 (Wardha Anjum)](#23-user-provisioning--user-2-wardha-anjum)
- [2.4 Full user roster & license cleanup](#24-full-user-roster--license-cleanup)
- [2.5 Security Group — assigned membership](#25-security-group--assigned-membership)
- [2.6 Microsoft 365 Group — auto-provisioned Team](#26-microsoft-365-group--auto-provisioned-team)
- [2.7 Dynamic Group — the membership-rule saga](#27-dynamic-group--the-membership-rule-saga)
- [2.8 Azure governance foundation](#28-azure-governance-foundation)
- [Summary of what this proves](#summary-of-what-this-proves)

---

## 2.1 Organization profile

Checked and confirmed the tenant's technical contact and directory identity before provisioning anything else.

![Organization information — tenant ID, technical contact](../assets/project-02-governance-foundation-full/01-org-profile-info.png)

---

## 2.2 User provisioning — User 1 (Md. Imran Ahmed)

Walked through the full "Add a user" wizard rather than the quick path, to see every step an admin is actually responsible for.

**Basics** — name and username:

![User basics filled in](../assets/project-02-governance-foundation-full/02-user1-basics.png)

**License assignment** — Business Premium:

![Business Premium license selected](../assets/project-02-governance-foundation-full/03-user1-license.png)

**Profile info** — `department: IT` set deliberately here, since it becomes the filter condition for the Dynamic Group built later in this project:

![Department set to IT](../assets/project-02-governance-foundation-full/04-user1-department-it.png)

**Review before finishing:**

![Review and finish screen](../assets/project-02-governance-foundation-full/05-user1-review.png)

**Confirmed:**

![Md. Imran Ahmed added to active users](../assets/project-02-governance-foundation-full/06-user1-confirmation.png)

---

## 2.3 User provisioning — User 2 (Wardha Anjum)

Second identity created with a **different** department (`Tech Support`) on purpose — this makes the Dynamic Group's filtering behavior meaningful to verify later (some users should match the rule, some shouldn't).

![Wardha Anjum review screen — Department: Tech Support](../assets/project-02-governance-foundation-full/07-user2-review-wardha.png)

Two more identities (Pallob Kumar Roy, Shakhawat Hossian Anik) were created the same way to round out a five-person test roster.

---

## 2.4 Full user roster & license cleanup

**Before:** one identity (Pallob Kumar Roy) still unlicensed after creation:

![Users list — Pallob unlicensed](../assets/project-02-governance-foundation-full/08-users-list-before-license.png)

**After:** license assigned, full roster licensed and ready:

![Users list — all five users licensed](../assets/project-02-governance-foundation-full/09-users-list-after-license.png)

Final roster: **Md. Imran Ahmed** (IT), **Wardha Anjum** (Tech Support), **Pallob Kumar Roy**, **Shakhawat Hossian Anik**, plus the admin account — a deliberately mixed set for the grouping exercises that follow.

---

## 2.5 Security Group — assigned membership

A standard, manually-managed group to scope access for an "IT Support" persona.

**Created:**

![Security group basics — IT-Support-Team](../assets/project-02-governance-foundation-full/10-security-group-basics.png)

![IT-Support-Team group created confirmation](../assets/project-02-governance-foundation-full/11-security-group-created.png)

**Verified final state** — one owner, two members:

![Security group owners and members confirmed](../assets/project-02-governance-foundation-full/12-security-group-members.png)

---

## 2.6 Microsoft 365 Group — auto-provisioned Team

Unlike a Security Group, a Microsoft 365 Group is meant to bootstrap a full collaboration space — the point of this task was to verify that actually happens, not just take it on faith.

**Created:**

![Microsoft 365 group basics — Managing-Team](../assets/project-02-governance-foundation-full/13-m365-group-basics.png)

**Members added:**

![Members added — Shakhawat, Wardha](../assets/project-02-governance-foundation-full/14-m365-group-members.png)

**Settings** — Privacy set to Private, and critically, **"Create a team for this group"** checked:

![Group settings — Create a Team for this group checked](../assets/project-02-governance-foundation-full/15-m365-group-settings-createteam.png)

**Verified:** the group shows a Teams-status icon in the group list, confirming the linked Team (and its SharePoint site) was actually provisioned, not just the bare group:

![Managing-Team in group list with Teams icon](../assets/project-02-governance-foundation-full/16-m365-group-final-list.png)

---

## 2.7 Dynamic Group — the membership-rule saga

This was the most instructive part of Project 2, because the first attempt **looked** successful but wasn't.

### The problem

A group named `Dynamic-IT-Department` was created through the Microsoft 365 admin center's standard "Add a security group" wizard. It looked complete — but had **zero owners and zero members**, with no way to add a membership rule anywhere in that wizard:

![Dynamic-IT-Department showing 0 owners, 0 members](../assets/project-02-governance-foundation-full/17-dynamic-group-empty-problem.png)

**Root cause:** the M365 admin center's simplified group wizard only supports **Assigned** membership. Dynamic (attribute-based) membership rules are configured in a different portal entirely — the **Entra admin center** (`entra.microsoft.com`).

### The fix

In the Entra admin center, the same group's **Properties** page exposes a **Membership type** dropdown with `Assigned` / `Dynamic User` / `Dynamic Device` options — not available in the M365 wizard at all:

![Membership type dropdown showing Dynamic User option](../assets/project-02-governance-foundation-full/18-entra-membership-type-dropdown.png)

Selecting **Dynamic User** opens a rule builder. Configured:

```
(user.department -eq "IT")
```

![Dynamic membership rule builder with syntax generated](../assets/project-02-governance-foundation-full/19-dynamic-rule-syntax.png)

Saved successfully, with Membership type now correctly showing **Dynamic User**:

![Group saved with Dynamic User membership type](../assets/project-02-governance-foundation-full/20-dynamic-group-saved.png)

### Verification

Within a few minutes, **Md. Imran Ahmed** (the only user with `department = IT`) was automatically added — no manual action taken:

![Entra ID showing Imran Ahmed auto-added as the sole matching member](../assets/project-02-governance-foundation-full/21-entra-member-auto-added.png)

**Note on propagation delay:** at the same moment, the Microsoft 365 admin center still reported "1 member" without showing the name — a visible reminder that Entra ID changes propagate to the M365 admin center on their own schedule, not instantly:

![M365 admin center showing member count but not yet synced](../assets/project-02-governance-foundation-full/22-m365-admin-syncing-delay.png)

---

## 2.8 Azure governance foundation

With the Microsoft 365 side complete, replicated the same governance mindset on the Azure side.

**Explored the hierarchy** — Tenant Root Group at the top, the subscription underneath it:

![Management groups showing Tenant Root Group and subscription](../assets/project-02-governance-foundation-full/23-azure-management-groups.png)

**Reviewed the subscription** — confirmed correct tenant (`BOL`), Owner role, $200 credit available, $0 spent so far:

![Subscription overview — directory, role, credit, cost](../assets/project-02-governance-foundation-full/24-azure-subscription-overview.png)

**Created a Resource Group** (`RG-Phase1-Lab`) as the scoping container every later Azure project (Storage, Networking, Compute) will be built inside:

![Resource group list showing new RG-Phase1-Lab](../assets/project-02-governance-foundation-full/25-resource-group-created.png)

**Applied tags** for cost tracking and organization before any resources were added to it:

![Resource group Tags page — environment: lab, owner: rakibul](../assets/project-02-governance-foundation-full/26-resource-group-tags.png)

---

## Summary of what this proves

| Area | What was built | What was learned |
|---|---|---|
| Identity | 5 test users with varied license/department attributes | Wizard-driven provisioning still requires deliberate attribute planning for later automation |
| Access (static) | Security Group, Microsoft 365 Group | M365 Groups auto-provision Teams + SharePoint — verified, not assumed |
| Access (dynamic) | Dynamic Group with attribute rule | The M365 admin center and Entra admin center are **not** functionally equivalent; dynamic rules live only in the latter |
| Timing | — | Directory changes (dynamic membership, cross-portal sync) are asynchronous; verify before assuming failure |
| Azure | Tagged Resource Group under the correct subscription | Governance (naming, tagging) is cheaper to apply at creation time than to retrofit |

**Next:** [Project 3 — Identity Security](./ROADMAP_DETAIL.md#project-3--identity-security-mfa-conditional-access)
