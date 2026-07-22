# Project 3 — Identity Security: MFA, Conditional Access & Identity Protection

[#project-3--identity-security-mfa-conditional-access--identity-protection](#project-3--identity-security-mfa-conditional-access--identity-protection)

**Goal:** layer per-user MFA, group-scoped Conditional Access, location-based access, and risk-based policies on top of the Project 1–2 tenant — then verify with logs that everything actually fires.

**Certification focus:** M365 Administration / SC-200

---

## What was built

| # | Item | Result |
|---|---|---|
| 1 | Per-user MFA | Enabled for test user `Shakhawat Hossian Anik` |
| 2 | CA01 — Group-based MFA | Reused existing `IT-Support-Team` group, required MFA for all cloud apps |
| 3 | CA02 — Location-based access | Created `Bangladesh-Trusted` named location, required MFA outside it |
| 4 | CA03 — Sign-in risk policy | Require MFA on Low/Medium/High sign-in risk |
| 5 | CA04 — User risk policy | Require MFA + password change on High user risk |
| 6 | Verification | 5 deliberate failed logins, confirmed via Sign-in logs + Policy impact |

All 4 CA policies were kept in **Report-only** — evaluated and logged, not enforced.

---

## Step 1 — Per-user MFA

1. Entra admin center → Users → **Per-user MFA**
2. Selected test user `Shakhawat Hossian Anik` → **Enable**

![Per-user MFA enabled](images/01_mfa_peruser_enabled.png)

---

## Step 2 — CA01: Require MFA for a group

1. Reused existing group `IT-Support-Team` (Security, Assigned) instead of creating a new one — added `Shakhawat` as a member
2. Conditional Access → New policy:
   - Users: `IT-Support-Team`
   - Target resources: All resources
   - Grant: Require MFA
   - State: **Report-only**

![CA01 created](images/03_ca_policy_created.png)

---

## Step 3 — CA02: Location-based access

1. Conditional Access → Named locations → created `Bangladesh-Trusted` (Countries)
2. New policy:
   - Network: Include = Any location · **Exclude = Bangladesh-Trusted**
   - Grant: Require MFA
   - State: **Report-only**

> ⚠️ First attempt put Bangladesh under *Include* by mistake — that would apply the policy only *inside* Bangladesh. Fixed by moving it to *Exclude*.

![CA01 + CA02 both listed](images/05_ca_policies_list_2.png)

---

## Step 4 — CA03 & CA04: Risk-based policies

The classic Identity Protection risk policy pages showed a banner saying they're being **retired Oct 1, 2026** — rebuilt both as Conditional Access policies instead.

**CA03 — Sign-in risk:**
- Condition: Sign-in risk = Low, Medium, High
- Grant: Require MFA

**CA04 — User risk:**
- Condition: User risk = High
- Grant: Require MFA **+** Require password change

> ⚠️ CA04 initially failed with *"All cloud apps" must be selected when "Require password change" grant is selected* — Target resources had defaulted to "All agent resources" instead of "All resources." Fixed by explicitly selecting All resources.

![All 4 policies, Report-only](images/06_all_4_ca_policies.png)

---

## Step 5 — Verify it actually works

1. Deliberately failed login 5x as `Shakhawat` with wrong password

![Failed login attempts](images/07_failed_login_attempts.png)

2. **Sign-in logs** confirmed all 5 attempts, error code `50126` (invalid username/password)

![Sign-in logs](images/08_signin_logs_failed_attempts.png)

3. **Conditional Access → CA01 → Policy impact** confirmed the policy matched these sign-ins — evaluated, logged, but not enforced (Report-only)

![Policy impact confirmed](images/09_policy_impact_report_only_confirmed.png)

**Chain confirmed end-to-end:** user → group → policy → log.

---

## Mistakes & fixes (quick reference)

| Mistake | Fix |
|---|---|
| Classic risk policy pages are retiring Oct 2026 | Rebuilt as Conditional Access policies |
| CA04 blocked target resources error | Selected "All resources" instead of "All agent resources" |
| CA02 location logic backwards | Moved Bangladesh from Include → Exclude |

---

## Key learnings

1. Report-only mode lets you confirm a policy fires correctly before it can lock anyone out.
2. Reuse existing groups when the fit is genuine — avoids duplicate scaffolding.
3. "Require password change" specifically requires "All resources" as the target.
4. Include vs. Exclude direction can flip a policy's entire effect.
5. Microsoft's own tooling is mid-migration — classic risk policies are being pushed toward Conditional Access.

---

**Next:** Project 4 — RBAC & Delegated Administration (M365 + Azure). Full roadmap: [`ROADMAP.md`](../ROADMAP.md)
