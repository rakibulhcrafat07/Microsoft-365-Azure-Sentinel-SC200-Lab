# Project 4 — RBAC & Delegated Administration (M365 + Azure)

**Goal:** move from "everyone is Global Admin / Owner" to scoped, least-privilege access — per-role M365 admin delegation, PIM just-in-time activation, Azure RBAC (built-in and custom), and a hard guardrail (Resource Lock) that survives even Owner-level access.

**Certification focus:** M365 Administration / AZ-104 / SC-200

---

## Why this is a long post

Most "I set up RBAC" write-ups show one role assignment and a green checkmark. This one shows the assignment silently landing as **Eligible** instead of **Active** twice in a row (Entra ID P2 extends PIM governance further than expected), a scoped admin whose user list *shrank* rather than throwing an error, a custom role that could operate on VMs but couldn't even open the resource group blade, and a resource lock that stopped a delete initiated by the tenant's own Global Administrator.

---

## What was built

| # | Item | Result |
| --- | --- | --- |
| 1 | M365 — User Administrator role | Assigned to a test user, verified: can reset passwords + create users, blocked from granting Global Admin |
| 2 | M365 — Administrative Unit | Scoped admin created for `IT-Support-AU`; verified the scoped admin's user list only shows in-scope members |
| 3 | M365 — PIM eligible role | Exchange Administrator set as Eligible (not Active); verified pre-activation lockout and self-service activation |
| 4 | AZ-104 — Azure RBAC Reader | Reader assigned on a resource group; verified read access + blocked resource creation |
| 5 | AZ-104 — Custom RBAC role | Built "VM Operator" (start/restart/stop only); verified action-level scoping and its blind spot |
| 6 | AZ-104 — Resource Lock | Delete lock on a resource group; verified it blocks deletion even for the Owner/Global Admin account |

---

## 4.1 — M365: User Administrator role

Entra admin center → Roles & administrators → **User Administrator** → Add assignments. Assigned to `Wardha Anjum` (not the admin account, so a misconfiguration here can't lock out the person configuring it) as an **Active** assignment.

![User Administrator role assigned](images/s02_useradmin_assigned_active.png)

Logged in as Wardha in a separate session — successfully signed into the M365 admin center.

![Wardha signed in to M365 admin center](images/s03_wardha_login_m365.png)

**Verified — could do:** reset another user's password,

![Password reset succeeded](images/s04_password_reset_success.png)

and create a brand-new user with full license assignment.

![New user created successfully](images/s05_new_user_created_success.png)

**Verified — could NOT do:** self-elevate to Global Administrator.

![Blocked from Global Admin — no permission to save changes](images/s06_global_admin_blocked.png)

> Textbook least privilege: full control over users and passwords, zero path to tenant-wide admin.

---

## 4.2 — M365: Administrative Unit (scoped admin)

Created `IT-Support-AU`, assigned `Shakhawat Hossian Anik` as User Administrator scoped to just this AU, then added two users as AU members.

![Administrative Unit created](images/s07_au_created_success.png)

![AU members added](images/s08_au_members_added.png)

> ⚠️ **Unexpected finding:** logged in as Shakhawat, the Active Users list itself only showed the 2 AU members — users outside the AU didn't appear in the list at all. Scoping isn't just an action restriction, it restricts *visibility* at the list level. Password reset succeeded for the in-scope user; there was nothing outside the AU to even attempt against.

![Scoped admin sees only AU members, password reset succeeds](images/s09_scoped_admin_limited_visibility.png)

---

## 4.3 — M365: PIM eligible role assignment

Assigned Exchange Administrator to a test user as **Eligible** (Permanently eligible), not Active — the point being that having the role assigned shouldn't mean having the power right away.

![Eligible assignment confirmed](images/s10_pim_eligible_assignment.png)

**Before activation:** the same user tried to open the M365 admin center and was blocked outright — eligible ≠ active.

![Blocked before activation](images/s11_before_activation_blocked.png)
> ⚠️ **Navigation note:** the self-service "Activate" button isn't under My Access/entitlement management — it's Entra ID → **Privileged Identity Management → My roles → Microsoft Entra roles**. Took several wrong turns to find it.

Self-activated with a 0.5-hour duration and a justification. All 3 stages passed.

![PIM activation completed successfully](images/s12_pim_activation_success.png)

Confirmed in the Active assignments tab — State: Activated, time-bound end time (not Permanent).

![Role now Activated, time-bound](images/s13_pim_role_activated_state.png)

**Verified with real access:** the same account now had full Exchange admin center access — mailboxes, mail flow, roles.

![Exchange admin center access after activation](images/s14_exchange_admin_access_after_activation.png)

---

## 4.4 — AZ-104: Azure RBAC — Reader role

Assigned built-in **Reader** on `RG-Phase1-Lab` to a test user via Access control (IAM).

> ⚠️ **Unexpected finding:** the assignment landed as **"Eligible time-bound"** instead of Active — this tenant's Entra ID P2 license extends PIM governance to Azure resource role assignments by default, not just Entra directory roles. Had to manually edit the assignment (Assignment type → Active, Permanent) to get a direct, non-PIM assignment.

![Reader landed as Eligible, then converted to Active Permanent](images/s16_reader_converted_active_permanent.png)

**Verified — could view:** logged in as the assigned user, opened the resource group directly and could see the Overview, tags, and subscription info.

![Reader can view the resource group](images/s17_reader_can_view_rg.png)

**Verified — could NOT create:** attempted to deploy a VM into the same resource group.

![Blocked creating a VM — resource provider registration denied](images/s18_reader_blocked_creating_vm.png)
> Blocked with: *"Resource provider(s) ... are not registered ... and you don't have permissions to register a resource provider."* Reader can see everything, write nothing.

---

## 4.5 — AZ-104: Custom RBAC role

Built a custom role, **VM Operator**, with exactly three Actions and nothing else:
- `Microsoft.Compute/virtualMachines/start/action`
- `Microsoft.Compute/virtualMachines/restart/action`
- `Microsoft.Compute/virtualMachines/powerOff/action`

![Custom role reviewed before creation](images/s19_custom_role_review_create.png)

![Custom role created successfully](images/s20_custom_role_created_success.png)

Assigned to a test user on `RG-Phase1-Lab` as Active Permanent (same PIM-default behavior as 4.4 showed up here too, and was corrected the same way).

![VM Operator assigned to a test user](images/s21_vm_operator_assigned.png)

> ⚠️ **Unexpected finding:** the assigned user got a **401** even just navigating directly to the resource group by URL — *"does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourceGroups/read'."* A role built purely from `Microsoft.Compute` actions doesn't implicitly include the generic `resourceGroups/read` permission needed to open the RG blade itself. Unlike built-in Reader (which carries a wildcard read across resource types), a narrowly-scoped custom role needs that permission added explicitly if the user is expected to browse to the resource through the portal rather than act on it via a known path (CLI, automation, etc.).

![401 — missing resourceGroups/read permission](images/s22_custom_role_401_resourcegroups_read.png)

---

## 4.6 — AZ-104: Resource Lock

Added a **Delete** lock on `RG-Phase1-Lab`.

![Resource lock added](images/s23_resource_lock_added.png)

**Verified — even as Owner/Global Admin:** attempted to delete the resource group directly from the same admin account used throughout this project.

![Delete blocked by lock, even for the account's own Owner role](images/s24_delete_blocked_by_lock.png)
> *"The resource group RG-Phase1-Lab is locked and can't be deleted."* Locks sit above RBAC entirely — no role assignment, however privileged, overrides one.

---

## Mistakes & fixes (quick reference)

| Mistake | Fix |
| --- | --- |
| New Azure role assignments (Reader, custom role) silently landed as "Eligible time-bound" instead of Active | Entra ID P2 extends PIM governance to Azure resource roles by default in this tenant — edited the assignment's Assignment type to Active + Permanent |
| Couldn't find the PIM self-service "Activate" button | It isn't under My Access (entitlement management) — it's under Entra ID → Privileged Identity Management → My roles → Microsoft Entra roles |
| Custom "VM Operator" role couldn't open the resource group blade despite valid VM-level permissions | Narrowly-scoped custom roles don't inherit `Microsoft.Resources/subscriptions/resourceGroups/read` — must be added explicitly if portal browsing is expected |

---

## Key learnings

1. **Least privilege is provable, not assumed** — every role in this project was verified two ways: what it *could* do, and what it explicitly could *not* do.
2. **Scoping restricts visibility, not just actions** — both the Administrative Unit and the custom role showed users/resources disappearing entirely from a scoped admin's view, rather than throwing an access-denied error.
3. **Eligible ≠ Active, and it's easy to get one when you meant the other** — this tenant's PIM-for-Azure-resources default caught out both the Reader and the custom role assignment.
4. **Built-in roles carry implicit permissions custom roles don't** — Reader's wildcard read access vs. a hand-built role needing `resourceGroups/read` spelled out is the clearest AZ-104-relevant gotcha from this project.
5. **A Resource Lock is the strongest guardrail in this whole toolkit** — it's the only control here that held even against the tenant's own Global Administrator / Owner.

---

**Next:** Full roadmap: [`ROADMAP.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/ROADMAP.md)
