# Project 3 — Identity Security: MFA, Conditional Access & Identity Protection

[#project-3--identity-security-mfa-conditional-access--identity-protection](#project-3--identity-security-mfa-conditional-access--identity-protection)

**Goal:** layer per-user MFA, group-scoped Conditional Access, location-based access, and risk-based policies on top of the Project 1–2 tenant — then verify with logs that every policy actually fires.

**Certification focus:** M365 Administration / SC-200

---

## Why this is a long post

Most "I enabled MFA" write-ups show a green checkmark. This one shows the templates flow that led nowhere, the location rule that was backwards on the first attempt, the grant control Entra refused to save, and the classic Identity Protection page that's quietly being retired. That's the part worth documenting.

---

## What was built

| # | Item | Result |
|---|---|---|
| 1 | Per-user MFA | Enabled and confirmed for test user `Shakhawat Hossian Anik` |
| 2 | Group decision | Reused existing `IT-Support-Team` instead of building a new one |
| 3 | CA01 — Group-based MFA | Require MFA for `IT-Support-Team`, all cloud apps |
| 4 | Named location | `Bangladesh-Trusted` (Countries/GPS) |
| 5 | CA02 — Location-based access | Require MFA outside Bangladesh |
| 6 | CA03 — Sign-in risk policy | Require MFA on Low/Medium/High sign-in risk |
| 7 | CA04 — User risk policy | Require MFA + password change on High user risk |
| 8 | Verification | 5 deliberate failed logins, confirmed via Sign-in logs + Policy impact |

All 4 Conditional Access policies were kept in **Report-only** throughout — evaluated and logged, never enforced.

---

## 3.1–3.2 — Per-user MFA

Starting point: the same 5 test users from Project 2.

![The 5 test users](images/s01_five_users.png)

Entra admin center → Users → Per-user MFA. All five start disabled.

![Per-user MFA page, all disabled](images/s02_mfa_page_all_disabled.png)

`Shakhawat Hossian Anik` selected — not the admin account, so a misconfiguration here can't lock out the person doing the configuring.

![Selecting Shakhawat](images/s03_mfa_select_shakhawat.png)

Entra's own confirmation dialog before enabling, with a note that unregistered users get redirected to `aka.ms/mfasetup`.

![Confirm enable dialog](images/s04_mfa_confirm_popup.png)

Confirmed: status flips to enabled for Shakhawat only.

![MFA enabled, confirmed](images/s05_mfa_enabled_confirmed.png)

---

## 3.3 — A decision point: build new, or reuse?

> The plan called for a fresh security group. But Project 2 already had `IT-Support-Team` — Security type, Assigned membership — sitting right there.

Checked existing groups first: 5 total (2 Security, 3 M365, 1 Dynamic).

![Groups overview](images/s06_groups_overview.png)

`IT-Support-Team`, before: 2 direct members — Md. Imran Ahmed and Pallob Kumar Roy.

![IT-Support-Team before](images/s08_itsupport_before_2members.png)

Shakhawat added — 3 group members found. No new group created; the existing one just grew by one.

![IT-Support-Team after](images/s09_itsupport_after_3members.png)

---

## 3.4 — CA01: require MFA for the group

**False start:** Conditional Access's "Create new policy from templates" only offers templates for all users or admins — not a specific group. Backed out and used the blank custom policy form instead.

Blank policy form — every field empty, built up one at a time.

![Blank CA policy form](images/s11_ca_blank_form.png)

Assigning `IT-Support-Team` as the target.

![Assigning IT-Support-Team](images/s12_ca01_assign_group.png)

Target resources set to "All resources (formerly All cloud apps)".

![Target resources: all cloud apps](images/s13_ca01_target_resources.png)

> ⚠️ A warning appeared saying Security Defaults might need disabling first. Since the policy was going in as **Report-only** — evaluated, never enforced — it was safe to proceed and revisit later.

![Security defaults warning](images/s14_ca01_security_defaults_warning.png)

CA01 created.

![CA01 created](images/s15_ca01_created_success.png)

Confirmed in the policy list: 1 out of 1, State: Report-only.

![CA01 confirmed in list](images/s16_ca01_confirmed_list.png)

---

## 3.5 — Named location: Bangladesh-Trusted

Named locations, empty — the prerequisite for any location-based policy.

![Named locations, empty](images/s17_named_locations_empty.png)

Defining Bangladesh as trusted (Countries lookup method).

![Configuring Bangladesh-Trusted](images/s18_named_location_bangladesh_config.png)

Saved — 1 named location found, not yet attached to any policy.

![Bangladesh-Trusted created](images/s19_named_location_created.png)

---

## 3.6 — CA02: location-based access

> ⚠️ **The mistake:** the first draft put `Bangladesh-Trusted` under **Include** — which would require MFA only *inside* Bangladesh, the opposite of the intent.

![Mistake: Bangladesh under Include](images/s20_ca02_mistake_include.png)

**Fixed:** Include = Any location, Exclude = `Bangladesh-Trusted`. MFA now required everywhere *except* the trusted country.

![Fixed: Bangladesh under Exclude](images/s21_ca02_fixed_exclude.png)

Confirmed: 2 out of 2 policies, both Report-only.

![CA01 and CA02 confirmed](images/s22_ca01_ca02_confirmed_list.png)

---

## 3.7 — CA03: sign-in risk policy

Opened Identity Protection's dashboard.

![Identity Protection dashboard](images/s23_id_protection_dashboard.png)

> **Unexpected finding:** the legacy Sign-in risk policy page showed a banner — this risk policy is now read-only and retires **October 1, 2026**. Microsoft wants risk signals migrated into Conditional Access, so that's exactly where CA03 and CA04 got built instead.

![Classic risk policy retiring notice](images/s24_signin_risk_retiring_notice.png)

Building CA03: condition set to Sign-in risk — Low, Medium, and High all checked, since the resulting action (an MFA prompt) is low-friction.

![Building CA03](images/s25_ca03_building.png)

Confirmed: 3 out of 3 policies.

![CA03 confirmed](images/s26_ca03_confirmed_list.png)

---

## 3.8 — CA04: user risk policy

> ⚠️ **The second mistake:** building CA04 with "Require password change" as a grant control, Entra refused to save — *"All cloud apps" must be selected when "Require password change" grant is selected.* Target resources had defaulted to "All agent resources" instead of "All resources." Fixed by explicitly selecting All resources.

![CA04 target resources error](images/s27_ca04_mistake_target_resources.png)

All four policies, Report-only — CA01 (group MFA), CA02 (location), CA03 (sign-in risk), CA04 (user risk).

![All 4 policies confirmed](images/s28_all_4_ca_confirmed.png)

---

## 3.9 — Verification: does it actually fire?

Deliberately failed login 5 times as `Shakhawat` with the wrong password.

![Failed login attempt](images/s29_failed_login_attempt.png)

**Sign-in logs** confirmed all 5 attempts, error code `50126` (invalid username/password), from Gulshan, Dhaka, BD.

![Sign-in logs confirmed](images/s30_signin_logs_confirmed.png)

**Conditional Access → CA01 → Policy impact** confirmed the policy matched these sign-ins — evaluated, logged, but not enforced (Report-only).

![Policy impact confirmed](images/s31_policy_impact_confirmed.png)

**Chain confirmed end-to-end:** user → group → policy → log.

---

## Mistakes & fixes (quick reference)

| Mistake | Fix |
|---|---|
| Conditional Access templates only target all users/admins, not a specific group | Used the blank custom policy form instead |
| Classic risk policy pages are retiring Oct 2026 | Rebuilt as Conditional Access policies (CA03/CA04) |
| CA02 location logic backwards (Bangladesh under Include) | Moved Bangladesh to Exclude, Include = Any location |
| CA04 blocked — "All cloud apps" required for password-change grant | Selected "All resources" instead of "All agent resources" |

---

## Key learnings

1. **Reuse existing groups when the fit is genuine** — avoids duplicate scaffolding.
2. **Report-only is the safe default** for new Conditional Access policies — confirms a policy fires correctly before it can lock anyone out.
3. **Include vs. Exclude direction can flip a policy's entire effect** — the same named location produces the opposite outcome depending on which side it sits.
4. **Grant controls have hard dependencies** — "Require password change" specifically requires "All resources" as the target.
5. **Microsoft's own tooling is mid-migration** — classic Identity Protection risk policies are being sunset in favor of Conditional Access.

---

**Next:** Project 4 — RBAC & Delegated Administration (M365 + Azure). Full roadmap: [`ROADMAP.md`](../ROADMAP.md)
