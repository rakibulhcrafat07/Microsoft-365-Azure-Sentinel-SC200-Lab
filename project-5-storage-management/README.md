# Project 5 — Azure Storage (AZ-104)
### Part of: Zero-to-SOC: A Self-Funded, Multi-Certification Cloud Security Lab

**Tenant:** BOL959.onmicrosoft.com | **Resource Group:** RG-Phase1-Lab | **Region:** East US
**Cost:** $0 (Azure free-tier trial credit)

---

## Objective

Hands-on practice with Azure Storage for the AZ-104 exam: creating a storage account, working with Blob containers, managing access tiers, mounting an Azure File Share as a network drive, generating a Shared Access Signature (SAS) token, configuring a lifecycle management policy, and testing soft delete recovery.

---

## What Was Built

| # | Task | Resource |
|---|---|---|
| 1 | Storage Account + Blob container upload/download | `strakibullab01`, container `mycontainer`, file `Test.txt` |
| 2 | Access tier change (Hot → Cool) | Blob-level tiering |
| 3 | Azure File Share + Windows map drive | `myfileshare` (SMB) |
| 4 | SAS token generation | Read-only, HTTPS-only, time-boxed URL |
| 5 | Lifecycle management policy | 30 days → Cool, 90 days → Archive, 365 days → Delete |
| 6 | Soft delete enable + recover | 7-day retention, delete + Undelete tested |

---

## Step-by-Step Walkthrough

### Task 1: Storage Account + Blob Container

1. Opened Azure Portal, searched **Storage accounts** → **+ Create**
   ![Azure Portal home](images/01_azure-portal-home.png)
   ![Search storage accounts](images/02_search-storage-accounts.png)
   ![Storage center — no accounts yet](images/03_storage-center-empty.png)

2. Configured account: name `strakibullab01`, Resource Group `RG-Phase1-Lab`, Region East US, Performance **Standard**, Redundancy **LRS**
   ![Create storage account form](images/04_create-storage-account-form.png)
   ![Deployment initializing](images/05_deployment-initializing.png)
   ![Storage account deployed successfully](images/06_storage-account-deployed.png)

3. Created a private Blob container `mycontainer`
   ![Containers list — default $logs only](images/07_containers-list-default.png)
   ![mycontainer created](images/08_mycontainer-created.png)

4. Uploaded a local test file and confirmed it appeared in the container
   ![Local Test.txt file](images/09_test-file-local.png)
   ![Upload blob panel](images/10_upload-blob-panel.png)
   ![Blob uploaded successfully](images/11_blob-uploaded-success.png)

---

### Task 2: Access Tier Change (Hot → Cool)

Blob storage offers three tiers — **Hot** (frequent access, higher storage cost), **Cool** (infrequent access, lower storage cost), and **Archive** (rarely accessed, cheapest storage, slow retrieval). Changed `Test.txt` from Hot to Cool via **Change tier**.

![Blob properties before tier change](images/12_blob-properties-before-tier-change.png)
![Change tier panel — Cool selected](images/13_change-tier-panel-cool-selected.png)
![Blob properties after — Cool confirmed](images/14_blob-properties-after-cool-tier.png)

---

### Task 3: Azure File Share + Map Drive

Azure Files provides SMB-based network shares that can be mounted like a normal Windows drive — different from Blob storage, which is accessed over HTTPS.

![Storage account overview / properties](images/15_storage-account-overview-properties.png)
![Classic file shares — empty](images/16_classic-file-shares-empty.png)

While creating the file share `myfileshare`, the wizard's **Backup** tab had **"Enable backup"** checked by default, which would have auto-provisioned a Recovery Services Vault. This was intentionally unchecked to avoid an unplanned resource.

![File share Backup tab — declined](images/17_file-share-backup-tab-declined.png)
![File share created — share URL visible](images/18_file-share-created.png)

Used Azure's **Connect** panel to generate the exact PowerShell script (cmdkey + New-PSDrive) and ran it locally.

![Connect panel + PowerShell mount success](images/19_connect-panel-and-powershell-mount-success.png)

**Issue found:** the drive mounted successfully in an **Administrator** PowerShell session, but did not appear in File Explorer (which runs as a normal user session).

![This PC — Z: drive not visible (issue)](images/20_this-pc-drive-not-visible-issue.png)

**Fix:** re-ran the same connect script in a normal (non-elevated) PowerShell window. The share then appeared correctly under **Network locations** in File Explorer.

![This PC — network location visible (fixed)](images/21_this-pc-network-location-fixed.png)

---

### Task 4: SAS Token Generation

A Shared Access Signature (SAS) grants time-boxed, permission-scoped access to a blob without sharing the storage account key.

![Generate SAS tab](images/22_test-file-generate-sas-tab.png)
![SAS token and URL generated](images/23_sas-token-and-url-generated.png)

Verified the SAS URL worked by opening it directly in a browser (no Azure login) — the file content loaded successfully within the token's validity window.

![SAS URL tested in browser](images/24_sas-url-tested-in-browser.png)

> **Note on confidentiality:** a SAS URL embeds a signed access token in its query string. Anyone holding the URL can access the resource until it expires — it should be handled like a temporary credential, not shared publicly.

---

### Task 5: Lifecycle Management Policy

Built a 3-tier automated lifecycle rule (`MoveToCoolRule`) to optimize storage cost over time:

- **30 days** since last modified → move to **Cool**
- **90 days** since last modified → move to **Archive** (with rehydrate-skip exception)
- **365 days** since last modified → **Delete**

![Lifecycle management — no rules yet](images/25_lifecycle-management-empty.png)
![Rule details tab](images/26_lifecycle-rule-details-tab.png)
![Base blobs tab — empty](images/27_lifecycle-rule-base-blobs-tab-empty.png)
![30-day → Cool condition set](images/28_lifecycle-rule-30days-cool.png)
![Full 3-tier policy: Cool → Archive → Delete](images/29_lifecycle-rule-full-3-tier-policy.png)
![Rule created and enabled](images/30_lifecycle-rule-created-success.png)

---

### Task 6: Soft Delete — Enable + Recover

Soft delete acts as a safety net for accidental deletions — deleted blobs remain recoverable for a retention window instead of being purged immediately.

![Data protection settings — soft delete enabled, 7-day retention](images/31_data-protection-soft-delete-enabled.png)

Deleted `Test.txt` to test the feature. The confirmation dialog explicitly confirmed it would move to a soft-deleted (recoverable) state rather than being purged.

![Delete confirmation — soft-deleted state warning](images/32_delete-confirmation-soft-delete-dialog.png)
![Blob successfully deleted — soft-deleted state](images/33_blob-successfully-deleted-soft-state.png)
![Storage account properties — soft delete confirmed active](images/34_storage-account-properties-confirmed.png)

The container's default view only shows **active** blobs, so the deleted file disappeared from the normal list.

![Container — only active blobs, 0 items](images/35_container-only-active-blobs-0-items.png)

Switched the filter to **"Show active and deleted blobs"** to locate the file in its deleted state, then used **Undelete** to recover it.

![Container — deleted blob visible with status](images/36_container-show-deleted-blobs-status.png)
![Undelete option in context menu](images/37_undelete-context-menu.png)
![Blob successfully undeleted — back to Current version](images/38_blob-successfully-undeleted.png)

---

## Mistakes & Fixes

| Mistake / Unexpected Behavior | Fix |
|---|---|
| Redundancy defaulted to **GRS** (Geo-redundant) during storage account creation, which is costlier than needed for a lab | Manually changed to **LRS** before creating the account |
| Creating a Classic File Share auto-checked **"Enable backup"**, which would have silently created a Recovery Services Vault | Manually unchecked "Enable backup" before finalizing the file share |
| Mounted file share (via `New-PSDrive`) was invisible in File Explorer despite PowerShell reporting success | The mount was done in an **Administrator** PowerShell session; Windows does not surface admin-session drive mappings to the normal-user File Explorer session. Re-ran the identical script in a **non-elevated** PowerShell window — the share then appeared correctly under Network locations |
| Deleted blob vanished from the container's default view, briefly looking like a permanent delete | Default view only shows active blobs — switched to **"Show active and deleted blobs"** to locate and recover it via **Undelete** |

---

## Key Learnings

- **Hot / Cool / Archive** access tiers are about trading storage cost against retrieval cost and speed — Archive is cheapest to store but requires rehydration before it can be read.
- **Blob storage** (HTTPS, object access) and **Azure File Share** (SMB, mountable drive) solve different problems and are configured in separate parts of the same storage account.
- **SAS tokens** provide scoped, time-limited access without ever exposing the storage account key — but the generated URL itself is sensitive and should be treated like a credential.
- **Lifecycle management** rules are evaluated in tier order (Hot → Cool → Archive → Delete); multiple conditions in one rule let a single policy fully automate a blob's cost lifecycle.
- **Soft delete** is not a UI illusion — deleted items truly persist server-side for the retention window and can be fully recovered via Undelete, including all original properties (tier, size, hash).
- Admin vs. non-elevated **PowerShell sessions have separate drive-mapping visibility** in Windows — a subtlety worth remembering for any future SMB/File Share troubleshooting.

---

## Roadmap Status

Part of the 16-project **Zero-to-SOC** roadmap (SC-200 + AZ-104), self-funded on free-tier Azure/M365 subscriptions. AWS equivalents for this phase are intentionally deferred to keep focus on Azure/M365.

Previous: Project 4 — RBAC & IAM (Azure RBAC, PIM, Administrative Units)
**Current: Project 5 — Azure Storage (this project)**
Next: Continuing through the remaining projects in the roadmap.
