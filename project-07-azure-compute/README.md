# Project 7 — Azure Compute (VM + AD DS + Defender for Identity)

**Certification focus:** AZ-104 / SC-200

**Status: COMPLETE.**

---

## Goal

Move from "storage and networking" into real compute — create both a Windows and a Linux VM, practice the everyday AZ-104 VM operations (resize, disks, snapshots, extensions, availability), and set up a Windows Server VM that becomes an Active Directory Domain Controller, generating real identity data for a Microsoft Defender for Identity sensor (SC-200 relevance).

AWS EC2 tasks from the original checklist were intentionally left out — this lab stays Azure/Microsoft-only, matching every earlier project.

---

## What's in this project

| # | Task | Status |
|---|---|---|
| 1 | Create Windows Server 2022 VM, fix RDP/NSG/Bastion access | ✅ Done |
| 2 | Create Ubuntu VM, verify SSH via Bastion | ✅ Done |
| 3 | Resize a running VM, hit a real regional quota limit | ✅ Done |
| 4 | Attach a data disk, initialize it in Windows, take a snapshot | ✅ Done |
| 5 | Install the Custom Script Extension, verify it actually ran | ✅ Done |
| 6 | Attempt a zone-pinned VM, hit a Portal quota-caching bug | ✅ Done (documented as a real limitation) |
| 7a | Install AD DS role and promote the server to a Domain Controller | ✅ Done |
| 7b | Install and register the Microsoft Defender for Identity sensor | ✅ Done (sensor reporting; 2 minor health issues documented as a follow-up) |
| 8 | Stop/deallocate both VMs to end the project cleanly | ✅ Done |

All screenshots referenced below live in the `images/` folder, numbered `001` to `143` in the exact order the work was done.

---

## Why this is a long, honest write-up

Nearly every step here hit a real snag before it worked — a VM size unavailable in this region, an NSG blocking RDP, then blocking Bastion for a different reason, a resize blocked by a shared regional vCPU quota, a Portal quota checker that flatly disagreed with the actual Quotas page, a Defender for Identity workspace that failed to provision on the first try, and a storage account that correctly refused a public download. None of these were hidden — they're documented step by step because that's what actually happened, and because the fixes are more useful to read than a clean "it just worked" story.

---

## Task 1 — Windows Server VM Setup

**Goal:** Create a Windows Server 2022 VM in the existing lab environment, and prove it's actually reachable — not just "created" in the portal, but something you can log into.

---

### Steps

**images/001_no_vms_yet.png** — Starting point: the Virtual Machines list is empty. Nothing has been deployed yet in this project.

**images/002_create_vm_basics_blank.png** — Opened "Create a virtual machine." By default Azure suggests an Ubuntu Linux image — this needed to be changed to Windows.

**images/003_image_selector_open.png** to **images/004_marketplace_search_results.png** — Searched the Marketplace for "windows server 2022" to find the official Microsoft image (not a third-party one).

**images/005_windows_server_2022_editions.png** — Multiple editions showed up. Picked **Windows Server 2022 Datacenter: Azure Edition — x64 Gen 2**, which is the full GUI version (not the lightweight "Core" edition, since AD DS setup later needs the graphical interface).

**images/006_basics_filled_size_error.png** — Filled in the resource group, VM name (`vm-win-addc`), and other Basics fields. The default size `Standard_B1s` showed a red error: **this size isn't available in East US for this subscription.**

**images/007_size_dropdown_no_b2s.png** to **images/009_d2s_v7_found.png** — Searched for an available size instead of guessing. Landed on `Standard_D2s_v7` (2 vCPUs, 8 GiB RAM) — confirmed available for this subscription and region.

**images/010_basics_tab_complete.png** — Basics tab fully filled: resource group `RG-Phase1-Lab`, VM name `vm-win-addc`, size `Standard_D2s_v7`, admin username `azureadmin`, a strong password, and inbound port RDP (3389) allowed.

**images/011_review_create_summary.png** and **images/012_networking_tab_summary.png** — Reviewed everything before creating. Networking tab confirmed the VM would join the VNet/subnet already built in Project 6 (`vnet-lab-sc200/subnet-app`), reusing existing infrastructure instead of creating a duplicate.

**images/013_deployment_complete.png** — Deployment succeeded. VM, network interface, and public IP were all created with status "OK".

**images/014_vm_overview_running.png** — VM confirmed Running, with its public IP (`20.51.164.218`) and private IP (`10.10.1.5`) visible.

---

### Trying to connect (this is where it got interesting)

**images/015_connect_menu_options.png** to **images/019_rdp_security_warning.png** — Downloaded the native RDP file from the Portal and opened it. Windows showed the normal "unknown publisher" security warnings — clicked through them as expected.

**images/020_rdp_connection_failed.png** — RDP failed anyway: *"Remote Desktop can't connect to the remote computer."*

**Why:** This VM was placed into the same subnet (`subnet-app`) used in Project 6, and that subnet already had a Network Security Group (`nsg-app-subnet`) attached — one that only allowed inbound SSH (port 22), not RDP (port 3389).

**images/021_subnet_nsg_shown.png** and **images/022_nsg_inbound_rules_ssh_only.png** — Confirmed the NSG only had an SSH rule.

**images/023_add_rdp_rule_form.png** and **images/024_rdp_rule_created.png** — Added a new inbound rule allowing port 3389 from "My IP."

**images/025_rdp_retry_security_warning.png** and **images/026_rdp_still_failed.png** — Tried RDP again. Still failed, with a different underlying cause this time.

**images/027_vm_overview_connect_menu.png** and **images/028_check_access_passed.png** — Used Azure's built-in "Check access" diagnostic on the Connect page. It reported the port **was** reachable from the source IP — meaning the NSG rule was working, but something else was blocking the connection.

**images/029_restart_vm_confirm.png** and **images/030_restart_success.png** — Restarted the VM in case the RDP listener hadn't fully initialized after the first boot.

**images/031_rdp_error_0x204.png** — Still failed, now with a specific error code `0x204`, which usually means the connection is being blocked before it even reaches the VM — most likely the local ISP or network blocking outbound RDP traffic.

**images/032_bastion_deploy_page.png** — Switched strategy entirely: deployed Azure Bastion (browser-based access, no RDP client and no port 3389 needed on the local side). Bastion had been deleted at the end of Project 6 to save cost, so it needed to be redeployed here.

**images/033_bastion_connect_form.png** and **images/034_bastion_connection_error.png** — First Bastion attempt also failed: *"The target machine encountered an error and has closed the connection."*

**Why:** Bastion reaches the VM from inside its own subnet (`AzureBastionSubnet`, `10.10.3.0/26`) — not from the user's own IP. The NSG rule from step 24 only allowed RDP from "My IP," so Bastion's traffic was still being blocked.

**images/035_add_bastion_rdp_rule.png** and **images/036_bastion_rule_created.png** — Added a second inbound rule allowing port 3389 specifically from the Bastion subnet range.

**images/037_bastion_still_failed.png** — One more failed attempt (Bastion sometimes needs a minute to fully "warm up" right after deployment).

**images/038_windows_desktop_success.png** — Success. The Windows Server 2022 desktop loaded in the browser via Bastion, confirming the VM is fully set up and reachable.


---

## Task 2 — Linux (Ubuntu) VM Setup

**Goal:** Create an Ubuntu VM and practice SSH-based access, to contrast against the Windows RDP experience in Task 1.

---

### Steps

**images/039_basics_filled.png** — Created a new VM `vm-ubuntu-lab` using image **Ubuntu Server 24.04 LTS**, size `Standard_D2s_v7` (same verified-working size from Task 1), password authentication, and inbound port SSH (22) allowed.

**images/040_networking_no_public_ip.png** — On the Networking tab, joined the same existing VNet/subnet (`vnet-lab-sc200/subnet-app`) as the Windows VM, but this time set **Public IP: None**. Since Bastion was already being used successfully for the Windows VM, there was no need to expose this VM to the public internet at all — a more secure setup than Task 1's public-IP approach.

**images/041_deployment_complete.png** — Deployment succeeded. Only a network interface was created (no public IP resource), confirming the no-public-IP choice took effect.

**images/042_bastion_ssh_success.png** — Connected via Bastion using SSH protocol instead of RDP. Landed directly at the `azureadmin@vm-ubuntu-lab:~$` prompt — no NSG troubleshooting needed this time, because Bastion's SSH path used a different mechanism than the RDP rule that had to be added for Windows.


---

## Task 3 — VM Resize

**Goal:** Change a running VM's size to a bigger one, and observe what actually happens during a resize (including regional quota limits — a real AZ-104 exam topic).

Resize was performed on the **Linux VM** (`vm-ubuntu-lab`) rather than the Windows VM, on purpose — the Windows VM is planned to become an Active Directory Domain Controller later in this project, so it's better to leave it untouched once its role is more critical.

---

### Steps

**images/043_vm_overview_before_resize.png** — Linux VM confirmed Running at its current size, `Standard_D2s_v7`.

**images/044_size_page_current_d2s_v7.png** — Opened the VM's Size page (Availability + scale → Size). A note here says resizing a running VM causes it to restart.

**images/045_resize_confirm_d4s_v7.png** — Selected `Standard_D4s_v7` (4 vCPUs, 16 GiB — double the current size) and confirmed the resize.

**images/046_resize_failed_quota.png** — Resize failed: *"exceeding the approved Total Regional Cores quota... Current Limit: 4, Current Usage: 4."* The subscription's D-series v7 family is capped at 4 vCPUs total in East US, and both VMs (2 vCPUs each) were already using all of it.

**images/047_stop_windows_vm_dialog.png** and **images/048_bastion_disconnected_after_stop.png** — Stopped (deallocated) the Windows VM to free up quota for the resize test.

**images/049_resize_failed_again_stale_quota.png** — Tried resizing again — same error, even though the Windows VM was now stopped. The Portal's quota counter had not refreshed yet (a caching delay, not a real quota problem).

**images/050_d4s_v5_not_available.png** — Tried a different VM family (`D4s_v5`) as an alternative. This family showed **"Request quota"** — meaning it isn't approved for this subscription at all, unrelated to the current usage issue.

**images/051_insufficient_quota_categories.png** — Checked the "Insufficient quota" categories on the Size page to see exactly which sizes were blocked and why.

**images/052_size_page_refreshed.png** — Refreshed the page and confirmed the current size was still `Standard_D2s_v7`.

**images/053_resizing_to_d2ds_v7.png** — Instead of chasing the D4s_v7 quota issue further, picked `Standard_D2ds_v7` — same 2 vCPU / 8 GiB tier as before, but a different disk-optimized variant, which was not blocked by the quota. Resize started successfully.

**images/054_vm_powering_off_for_resize.png** — Watched the resize happen live through the Bastion terminal session: the VM itself printed *"The system will power off now!"* — direct proof that resizing triggers a reboot, exactly as the earlier warning said.

**images/055_reconnected_after_resize.png** — Reconnected via Bastion after the resize. The login banner showed a fresh session (`Last login... from 10.10.3.4`, the Bastion subnet's IP), confirming the VM came back up successfully at its new size.

---

### Outcome

The resize from `D2s_v7` → `D2ds_v7` succeeded and demonstrated the resize mechanics (restart-on-resize) clearly. The larger upsize to `D4s_v7` was blocked by a real regional quota limit shared across every VM in the subscription — a genuine constraint of running on a free-trial subscription, not a configuration mistake.


---

## Task 4 — Managed Disk & Snapshot

**Goal:** Attach an extra data disk to the Windows VM, bring it online inside Windows, and take a snapshot (a point-in-time backup) of that disk.

Done on the **Windows VM** (`vm-win-addc`) on purpose — in a real Active Directory setup, the AD database, logs, and SYSVOL folder are always kept on a separate data drive rather than the OS drive. This disk will become that data drive once AD DS is configured later in this project.

---

### Steps

**images/056_bastion_disconnected_vm_stopped.png** — The Windows VM had been stopped earlier (Task 3) to free up quota, so it needed to be started again before this task.

**images/057_vm_agent_not_ready_after_start.png** — On the first connection attempt after starting the VM, the Portal showed a warning that the VM agent wasn't ready yet — the VM needs a short warm-up period right after starting.

**images/058_windows_desktop_reconnected.png** — Second attempt succeeded — back on the Windows desktop via Bastion.

**images/059_vm_list_both_running.png** — Confirmed both VMs (Windows and Linux) were Running before continuing.

**images/060_disks_page_empty.png** — Opened the Windows VM's Disks page. Only the OS disk existed; "Data disks: 0."

**images/061_new_disk_form_filled.png** — Clicked "Create and attach a new disk" and filled in: name `vm-win-addc-datadisk01`, storage type **Standard SSD** (cheaper than Premium, fine for a lab), size **32 GiB**.

**images/062_disk_created_attached.png** — Clicked Apply. Two success notifications appeared: "Successfully created disk" and "Updated virtual machine" — the disk now shows in the Data disks list at LUN 0.

**images/063_initialize_disk_dialog.png** — Opened Disk Management inside Windows (`diskmgmt.msc`). Azure automatically prompted to **Initialize Disk** for the new 32 GB disk, which showed as "Not Initialized." Chose **MBR** partition style (simpler, and more than enough for a 32 GB disk — GPT is only needed for disks over 2 TB or special boot scenarios).

**images/064_disk_management_online.png** — After initializing and creating a New Simple Volume (formatted NTFS, drive letter **D:**, labeled "Disk Addition"), Disk Management showed **Disk 1: 32.00 GB, Online, Healthy (Primary Partition)**.

**images/065_file_explorer_d_drive.png** — Confirmed in File Explorer: "Disk Addition (D:)" now appears as a normal, usable drive with 31.9 GB free.

---

### Snapshot

**images/066_disk_resource_overview.png** — Opened the disk's own resource page in the Portal (not the VM's Disks tab, but the Disk resource itself) and clicked "Create snapshot."

**images/067_create_snapshot_form.png** — The snapshot form auto-filled the source disk. Kept the defaults: **Incremental** snapshot type (only stores the difference from the last snapshot, cheaper) and **Standard HDD** storage (cheapest, fine for a backup that isn't accessed often). Named it `vm-win-addc-datadisk01-snapshot01`.

**images/068_snapshot_deployment_complete.png** and **images/069_snapshot_deployment_details.png** — Snapshot deployment succeeded, confirmed with status "OK" in the deployment details.

---

### Outcome

The data disk is attached, initialized, formatted, and usable inside Windows as drive D:. A snapshot of that disk now exists as a recoverable backup point — both pieces of the task completed and verified.


---

## Task 5 — VM Extension (Custom Script)

**Goal:** Install a VM extension (a small agent that runs automation inside the VM after deployment) and prove that it actually executed — not just that Azure reported success.

Used the **Custom Script Extension**, which downloads and runs a PowerShell script inside the VM. Installed on the Windows VM (`vm-win-addc`).

---

### Steps

**images/070_extensions_page_empty.png** — Opened the VM's "Extensions + applications" page. Nothing installed yet ("No resource extensions found").

**images/071_install_extension_gallery.png** — Clicked "+ Add" to browse the extension gallery — dozens of options are available (monitoring agents, security tools, automation agents, etc.).

**images/072_custom_script_extension_card.png** and **images/073_custom_script_extension_selected.png** — Selected **Custom script extension** (published by Microsoft Corp.). Its description: it can download PowerShell scripts and files from Azure storage and run them on the VM, useful for post-deployment automation.

**images/074_configure_extension_form.png** — The configuration screen asks for a script file. Note: "Browse" here does **not** browse the local computer — it browses an Azure Storage Account, since the extension needs to fetch the script from Azure, not from a local file.

**images/075_powershell_script_file_icon.png** and **images/076_powershell_script_content.png** — Wrote a simple test script locally, saved as `install-extension-test.ps1` (had to choose "All Files" when saving in Notepad so it wouldn't default to `.txt`). The script creates a folder and a text file with a timestamp — a simple, verifiable proof that the script actually ran.

```powershell
New-Item -Path "C:\ExtensionTest" -ItemType Directory -Force
Set-Content -Path "C:\ExtensionTest\extension-proof.txt" -Value "Custom Script Extension ran successfully on $(Get-Date)"
```

**images/077_storage_accounts_browse.png** and **images/078_storage_containers_list.png** — Browsed to the storage account already built in Project 5 (`strakibullab01`) and its existing `mycontainer` blob container, reusing infrastructure from an earlier project instead of creating a new one.

**images/079_script_uploaded_to_blob.png** — Uploaded `install-extension-test.ps1` into the container. It now sits alongside `Test.txt` from Project 5.

**images/080_review_create_extension.png** — Reviewed the extension configuration (script file path filled in automatically after selecting the uploaded blob) and clicked Create.

**images/081_extension_deployment_succeeded.png** — Deployment succeeded.

---

### Verifying it actually ran

**images/082_extension_test_folder_created.png** — Connected to the VM via Bastion and opened `C:\ExtensionTest` in File Explorer. The file `extension-proof.txt` was there, dated at the exact time of deployment.

**images/083_extension_proof_file_content.png** — Opened the file in Notepad. It read: *"Custom Script Extension ran successfully on 07/31/2026 13:17:02"* — solid proof the script executed inside the VM, not just that Azure reported "success" in the Portal.


---

## Task 6 — Availability Zone (Attempted)

**Goal:** Deploy a VM pinned to a specific Availability Zone, to understand how zone placement works in Azure.

Availability Zone has to be chosen **at VM creation time** — it can't be added to an already-running VM afterward. Since both existing VMs (Windows and Linux) were already using the full regional vCPU quota, a brand-new demo VM (`vm-az-demo`) was needed to test this feature.

---

### Steps

**images/084_demo_vm_basics_quota_error.png** — Started creating `vm-az-demo` with **Availability options: Availability zone**, **Zone 1** selected. The Size field showed a red error: *"2 vCPUs are needed for this configuration, but only 0 vCPUs (of 4) remain in your subscription."*

**images/085_linux_vm_stopped_to_free_quota.png** — Stopped (deallocated) the Linux VM to free up 2 vCPUs of quota, expecting the new VM to then have room.

**images/086_quota_error_still_showing.png** — Tried again (including refreshing the quota and starting the wizard fresh) — same "0 of 4 remain" error persisted.

**images/087_quotas_page_actual_usage_2of4.png** — Checked the actual subscription-level Quotas page (Usage + quotas → Standard Dsv7 Family vCPUs, East US). It showed **"2 of 4"** in use — meaning 2 vCPUs should have been free, directly contradicting what the VM Create wizard was reporting.

---

### Outcome

This is a genuine discrepancy between the Portal's VM Create wizard (which checks quota through one code path) and the actual Quotas service (a different, more authoritative source). Rather than force a workaround — for example, deploying via Azure CLI to bypass the wizard's stale check — this was documented honestly as a real tooling limitation encountered while working within a free-trial subscription.

The Availability Zone **concept** was still explored through the Basics tab during this attempt: the "Zone options" setting offers a choice between **Self-selected zone** (pick Zone 1, 2, or 3 yourself) and **Azure-selected zone** (let Azure pick the best zone automatically) — both options were visible and understood, even though the deployment itself couldn't complete due to the quota issue above.


---

## Task 7a — Active Directory Domain Services (AD DS)

**Goal:** Install the Active Directory Domain Services role on the Windows VM and promote it to a fully functioning Domain Controller. This generates the real identity data (users, logins, group changes) that a Microsoft Defender for Identity sensor needs to analyze — the main reason this VM exists in the project.

**Status: Complete.** The AD DS role was installed, the server was promoted to the first Domain Controller of a brand-new forest (`rakibul.local`), and the promotion was verified by confirming the Kerberos Key Distribution Center service is running (a service that only runs on an actual domain controller). See `08-defender-for-identity-sensor/` for the next part — installing the Defender for Identity sensor on this now-functioning Domain Controller.

---

### Steps

**images/088_server_manager_dashboard.png** — Opened Server Manager on the Windows VM (opens automatically, or from the taskbar). Closed an informational popup suggesting Windows Admin Center/Azure Arc as alternatives — Server Manager was used instead since it directly matches the AZ-104 exam objectives.

**images/089_add_roles_wizard_start.png** — From Server Manager's "Manage" menu, opened **Add Roles and Features Wizard**. The "Before You Begin" page confirmed the destination server as `vm-win-addc`.

**images/090_installation_type.png** — Selected **Role-based or feature-based installation** (the standard option for configuring a single server, as opposed to Remote Desktop Services installation).

**images/091_server_selection.png** — Confirmed `vm-win-addc` (10.10.1.5) as the target server from the server pool.

**images/092_server_roles_ad_ds_checked.png** — Checked **Active Directory Domain Services** in the roles list. The description on the right explained: AD DS stores information about network objects (users, computers) and lets administrators manage that information and control access across the network.

**images/093_features_page.png** — On the Features page, **Group Policy Management** was already automatically checked — it's a required companion tool for AD DS, added automatically as a dependency. Nothing else needed to be selected.

**images/094_ad_ds_info_page.png** — An informational page about AD DS appeared, noting that production environments should have at least two domain controllers for redundancy (not required for this single-server lab).

**images/095_confirmation_page.png** — Reviewed the final list of what would be installed: Active Directory Domain Services, Group Policy Management, Remote Server Administration Tools, AD DS and AD LDS Tools, Active Directory module for PowerShell, AD DS Tools, Active Directory Administrative Center, and AD DS Snap-Ins.

**images/096_installation_progress_succeeded.png** — Installation completed: **"Configuration required. Installation succeeded on vm-win-addc."** A link appeared: **"Promote this server to a domain controller"** — this is the next required step, since installing the role alone does not make the server a functioning Domain Controller yet.

**images/097_server_manager_ad_ds_added.png** — Back on the Server Manager Dashboard, a new **AD DS** section now appears in "Roles and Server Groups" (Roles count went from 1 to 2), confirming the role is present on the server.

---

### Promoting the server to a Domain Controller

**images/098_promote_to_dc_notification.png** — Returned to the Server Manager Dashboard notification flag and clicked **"Promote this server to a domain controller"** to launch the AD DS Configuration Wizard.

**images/099_deployment_config_default_wrong.png** — The wizard's **Deployment Configuration** screen defaulted to "Add a domain controller to an existing domain" — the wrong option, since no domain exists yet on this server.

**images/100_deployment_config_new_forest.png** — Switched to **"Add a new forest"** and entered the root domain name `rakibul.local`. This is the correct choice for the very first domain controller — it creates a brand-new Active Directory forest from scratch.

**images/101_domain_controller_options.png** — On **Domain Controller Options**, left the Forest/Domain functional level at the default "Windows Server 2016," kept DNS Server and Global Catalog checked, and set a Directory Services Restore Mode (DSRM) recovery password.

**images/102_dns_options_warning.png** — The **DNS Options** screen showed a yellow warning that a DNS delegation could not be created. This is expected and harmless — it only matters when a domain is meant to be a child of an existing parent DNS zone, which isn't the case for this isolated lab forest.

**images/103_additional_options_netbios.png** — The wizard auto-generated the NetBIOS domain name `RAKIBUL` on the **Additional Options** screen — left as-is.

**images/104_paths_default.png** — Left the AD DS database, log files, and SYSVOL folder locations at their Windows defaults on the **Paths** screen.

**images/105_review_options_summary.png** — The **Review Options** screen summarized the whole configuration: new forest, domain `rakibul.local`, NetBIOS `RAKIBUL`, DNS Server and Global Catalog both enabled.

**images/106_prerequisites_check_passed.png** — **Prerequisites Check** passed with a green "All prerequisite checks passed successfully" message. Two informational (non-blocking) notes appeared about cryptography compatibility settings and static IP recommendations — both normal for a lab VM. Clicked **Install**.

**images/107_connection_error_during_reboot.png** — Immediately after clicking Install, the Bastion remote session dropped with a "Connection Error" dialog. This is expected — promoting a server to a domain controller always ends with an automatic reboot, which disconnects any active remote session. Waited a few minutes before reconnecting.

**images/108_reconnected_ad_ds_dns_roles.png** — After reconnecting via Bastion, Server Manager's Dashboard now listed **both AD DS and DNS** as active roles (previously only AD DS, without DNS) — the first sign the promotion succeeded.

**images/109_services_ntds_kdc_running.png** — Opened the AD DS section in Server Manager and checked the **Services** panel: **"Active Directory Domain Services" (NTDS)** and **"Kerberos Key Distribution Center" (Kdc)** were both **Running**. The KDC service only runs on an actual, working domain controller — this is the clearest proof the promotion completed successfully.

---

### Mistakes & fixes (promotion phase)

| # | What happened | Explanation / fix |
|---|---|---|
| 1 | Bastion session dropped with "Connection Error" right after clicking Install | Expected — a domain controller promotion always ends in an automatic reboot, which drops any active remote session. Waited and reconnected. |

### Key learnings (promotion phase)

- Installing the AD DS **role** (this README's first half) and **promoting** the server to an actual domain controller (this README's second half) are two separate steps — the role alone does nothing until promotion runs.
- A DNS delegation warning during promotion is normal for a standalone/isolated lab forest with no parent DNS zone — it's informational, not an error.
- The Kerberos Key Distribution Center (KDC) service is the most reliable proof a promotion succeeded, since it only runs on genuine domain controllers.

### What's next

- Install the **Microsoft Defender for Identity sensor** on this now-functioning Domain Controller — see `08-defender-for-identity-sensor/`
- Generate some real activity (a new AD user, a login) so Defender for Identity has actual data to work with, setting up later SC-200 detection projects


---

## Task 7b — Microsoft Defender for Identity Sensor

**Goal:** Install the Microsoft Defender for Identity (MDI) sensor on `vm-win-addc`, now that it's a working Domain Controller, so it can monitor the on-premises Active Directory environment for suspicious activity — the SC-200-relevant reason this VM exists.

**Status: Complete.** The sensor is installed and registered in the Defender portal, reporting from domain `rakibul.local`. It shows two open health issues after this first install, which are normal post-install items and are documented below rather than hidden.

---

### Part 1 — Finding the right page in the Defender portal

This part took several wrong turns before landing on the correct settings page — all of them are documented here honestly, since dead ends and how to recover from them are as useful to read as the steps that worked.

**images/110_defender_portal_home.png** — Opened `security.microsoft.com`, the unified Microsoft Defender portal, to begin locating Defender for Identity sensor settings.

**images/111_identities_menu_expanded.png** — Expanded the **Identities** menu in the left sidebar to see its sub-pages: Dashboard, Coverage & Maturity, Service accounts, Password protection, Tools.

**images/112_wrong_turn_defender_for_business.png** — Clicked "Tools" under Identities — this actually opened a generic "Welcome to Microsoft Defender for Business" onboarding screen, unrelated to sensor management. Wrong turn #1.

**images/113_identity_tools_page.png** — The *correct* Identities → Tools page (Documentation, Sizing Tool, Readiness Script, PowerShell module) — confirmed MDI licensing was active, but still not the sensor download page.

**images/114_wrong_turn_onboarding_wizard.png** — Searching "sensors" in the top search bar accidentally led into a **Defender for Business onboarding wizard** ("Let's give people access") — an unrelated setup flow. Wrong turn #2, cancelled.

**images/115_back_to_home.png** — Returned to the Defender portal Home page to try a different navigation path: **System → Settings** instead of the Identities menu directly.

**images/116_settings_hub_identities_link.png** — Found the correct **Settings** hub, listing all setting categories including **"Identities"** ("General settings for identities") — this was the right link all along.

---

### Part 2 — First-time workspace provisioning

**images/117_workspace_provisioning_loading.png** — Clicking "Identities" here triggered a one-time setup screen: **"Hang on! We're preparing your Microsoft Defender for Identity workspace."** This confirmed the MDI workspace is provisioned lazily on first access, not automatically the moment the license is assigned.

**images/118_workspace_provisioning_failed.png** — Provisioning failed with **"Something went wrong — MDI instance could not be created."** Rather than assume misconfiguration, this was checked methodically:

**images/119_license_check_ems_e5.png** — Verified in the Microsoft 365 admin center that the correct license (**Enterprise Mobility + Security E5**, which includes Defender for Identity) was assigned to the signed-in account — ruled out a licensing problem.

**images/120_global_admin_role_confirmed.png** — Verified in the Entra admin center that the account held an **active** Global Administrator role (not just "eligible," which would have needed PIM activation) — ruled out a permissions problem.

With both licensing and permissions confirmed correct, the failure was treated as a temporary backend delay on Microsoft's side. Retried after a short wait.

**images/121_identity_security_page_loaded.png** — On retry, the **Identity Security** page loaded successfully, showing Activation / Sensors / Service Accounts Classification tabs. (The Activation tab's server list briefly still showed "Failed to load data" — a separate, smaller issue, addressed below.)

---

### Part 3 — Choosing an install method

**images/122_sensors_tab_none_installed.png** — The **Sensors** tab loaded correctly, showing **"No sensors installed."** The workspace was now ready, just empty.

**images/123_add_sensor_dialog_two_options.png** — Clicked **"Add sensor."** A dialog offered two paths: the newer **"Activate servers"** method (works only for servers already onboarded into Defender for Endpoint's device inventory) and the older **"Continue with the classic sensor"** method (a manual downloadable installer).

**images/124_activation_tab_failed_to_load.png** — Tried the Activation tab's server list again — it kept showing "Failed to load data," which turned out to be because `vm-win-addc` was never onboarded to Defender for Endpoint, so the newer activation method had nothing to list.

**images/125_classic_sensor_access_key_panel.png** — Switched to **"Continue with the classic sensor"** instead. This opened a panel with a **"Download installer"** button and a generated **Access key** — everything needed for a manual install, with no Defender for Endpoint dependency.

---

### Part 4 — Getting the installer onto the Domain Controller

Bastion sessions don't support drag-and-drop or clipboard file transfer for actual files, so the same technique from Task 5 (Custom Script Extension) was reused: upload to Blob Storage, then download from inside the VM.

**images/126_upload_installer_to_storage.png** — Uploaded the downloaded `Azure ATP Sensor Setup.zip` (84.55 MiB) to the existing Project 5 storage account (`strakibullab01` → `mycontainer`) — reusing infrastructure across projects rather than creating something new.

**images/127_blob_uploaded_confirmed.png** — Upload completed; the container now listed three files, including the new sensor zip alongside earlier test files.

**images/128_bastion_connect_panel.png** — Opened a fresh Bastion connection to `vm-win-addc`. Username/password were already saved from the earlier session.

**images/129_public_access_error_xml.png** — Tried downloading the zip directly inside the VM using the plain blob URL — got an **XML error: `PublicAccessNotPermitted`**. This confirmed the storage account correctly blocks anonymous public access by default (the same secure-by-default behavior explored back in the Storage project).

**images/130_generate_sas_token.png** — Generated a **SAS (Shared Access Signature) token** for the blob — Read-only permission, HTTPS-only, same-day expiry — the proper, time-limited way to allow a private download without opening the storage account to the whole internet.

**images/131_sas_download_extracted_in_vm.png** — Pasted the SAS URL into the VM's browser. The download succeeded, and the zip was extracted, revealing three items: an `NPCAP` driver folder, the `Azure ATP Sensor Setup` application, and a `SensorInstallationConfiguration.json` file.

**images/132_downloads_folder_zip_and_extracted.png** / **images/133_extracted_folder_contents.png** — Confirmed both the downloaded zip and its extracted contents were present and correct before running the installer.

---

### Part 5 — Installing the sensor

**images/134_installer_wizard_language.png** — Ran the installer. The **Microsoft Defender for Identity** setup wizard launched (English selected).

**images/135_sensor_deployment_type_selected.png** — On **Sensor deployment type**, selected **"Sensor"** (installed directly on the domain controller) — the correct choice, since `vm-win-addc` is itself the domain controller, as opposed to "Standalone Sensor" (needs port-mirroring) or the AD FS/Entra Connect option.

**images/136_configure_sensor_empty_key.png** — The **Configure the Sensor** screen appeared with an empty Access key field and the default installation path.

**images/137_configure_sensor_key_pasted.png** — Pasted the Access key generated earlier (clipboard sync worked through the Bastion session).

**images/138_installation_progress.png** — Installation in progress, showing both a per-component progress bar and an overall progress bar.

**images/139_installation_completed.png** — **"Installation completed successfully"** with a green checkmark.

---

### Part 6 — Verifying the sensor

**images/140_sensor_registered_not_healthy.png** — Back in the Defender portal's Sensors tab, `vm-win-addc` now appeared as a registered sensor — Type: Domain controller, Domain: `rakibul.local`, Version 2.255.19295.47272 — but with Health status **"Not healthy"** and 2 open issues.

**images/141_health_issues_detail.png** — Opened the health issue details: **"NTLM Auditing is not enabled"** and **"Services Advanced Auditing is not enabled,"** both Medium severity. These are common first-install issues that need Group Policy changes (Advanced Audit Policy Configuration) to resolve — not something the installer configures automatically. Documented here as an honest finding rather than fixed immediately, since the core goal (sensor installed and reporting to the portal) was achieved.

---

### Mistakes & fixes

| # | What happened | Fix / explanation |
|---|---|---|
| 1 | Spent several steps clicking through the wrong Defender portal pages (Identities → Tools, a search bar detour, a Defender for Business onboarding wizard) trying to find the sensor page | The correct path is **System → Settings → Identities**, not the Identities menu's own "Tools" item. |
| 2 | MDI workspace provisioning failed the first time ("MDI instance could not be created") | Verified license (EMS E5) and role (active Global Administrator) were both correct — this was a temporary backend delay on Microsoft's side. Retried later and it succeeded. |
| 3 | The newer "Activate servers" sensor method never listed the server | It depends on the server being onboarded to Defender for Endpoint's device inventory, which `vm-win-addc` wasn't. Used the classic downloadable installer instead, which has no such dependency. |
| 4 | Direct blob URL download inside the VM failed with `PublicAccessNotPermitted` | Storage account correctly blocks public access by default. Generated a time-limited SAS token instead — the secure, intended way to share a private blob temporarily. |
| 5 | Sensor showed "Not healthy" with 2 open issues right after install | Normal for a first install. The issues (NTLM Auditing, Advanced Auditing not enabled) need Group Policy configuration — logged as a follow-up item rather than blocking task completion. |

### Key learnings

- A Defender for Identity workspace is provisioned **lazily**, on first access — not automatically the instant a license is assigned. First-time access can fail transiently and just needs a retry.
- The newer "Activate servers" sensor flow silently depends on Defender for Endpoint device inventory data; the classic manual installer has no such dependency and is more reliable for a lab where not every server is onboarded to Endpoint.
- Secure-by-default storage (no public blob access) applies even when moving files for internal lab purposes — SAS tokens are the correct way to do this without disabling storage account security.
- A brand-new sensor install is expected to show open health issues; "installed and reporting" and "fully healthy" are two different milestones.

### Infrastructure reused from earlier projects

- **Storage account `strakibullab01` / `mycontainer`** (Project 5) — reused again to move the sensor installer into the VM, the same pattern used for the Custom Script Extension in Task 5.

### What's next

- (Optional, future task) Configure NTLM Auditing and Advanced Audit Policy via Group Policy to bring the sensor to a fully "Healthy" status.
- Generate real AD activity (new users, logins) so Defender for Identity has data to analyze for later SC-200 detection projects.


---

## Task 8 — VM Stop / Deallocate

**Goal:** Shut down both lab virtual machines properly at the end of the project so they stop consuming subscription credit and regional vCPU quota while not in active use.

In Azure, simply shutting down a VM's operating system from inside Windows/Linux is not enough to stop billing — the VM has to be **Stopped (Deallocated)** from the Azure control plane itself, which releases the underlying compute hardware.

---

### Steps

**images/142_vm_win_addc_stopped.png** — Selected `vm-win-addc` in the Azure Portal and clicked **Stop**. The VM's Status changed to **"Stopped (deallocated)"**, and a "Successfully stopped virtual machine" notification confirmed it.

**images/143_vm_ubuntu_lab_stopped.png** — Repeated the same action for `vm-ubuntu-lab`. Its Status also shows **"Stopped (deallocated)"**, with a matching success notification.

Both VMs retain their disks, configuration, and network settings while stopped — they can be started again later without losing any of the work done in this project (including the AD DS forest and the installed Defender for Identity sensor).

---

### Key learnings

- "Stopped (deallocated)" is different from just shutting down the OS — only deallocation actually releases the compute resources and stops per-hour billing.
- Stopping a VM does not delete anything — disks, NICs, and configuration are preserved, so the domain controller and sensor setup will still be there the next time the VM is started.


---

## Full mistakes & fixes list (all tasks)

| # | Task | What went wrong | Fix / explanation |
|---|---|---|---|
| 1 | 1 | `Standard_B1s` size unavailable when creating the Windows VM | Switched to `Standard_D2s_v7`, which was available in this subscription/region |
| 2 | 1 | RDP failed even after downloading the RDP file | The VM's subnet (`subnet-app`, reused from Project 6) had an NSG that only allowed SSH, not RDP — added an inbound rule for port 3389 from "My IP" |
| 3 | 1 | RDP still failed after the NSG fix (error `0x204`) | Local ISP/network was blocking outbound RDP entirely — switched to Azure Bastion (browser-based, no local RDP client needed) |
| 4 | 1 | Bastion also failed on the first try | Bastion traffic comes from the `AzureBastionSubnet` range, not the user's IP — added a second NSG rule allowing RDP specifically from the Bastion subnet |
| 5 | 3 | VM resize to `Standard_D4s_v7` failed | Regional vCPU quota for the D-series v7 family was fully used by both VMs together — stopped one VM to free quota, then used `Standard_D2ds_v7` instead |
| 6 | 3 | Resize kept failing with the same quota message even after stopping a VM | The Portal's size picker shows a cached quota number — waited, then picked a size within the confirmed available quota |
| 7 | 6 | A zone-pinned demo VM (`vm-az-demo`) could not be created despite quota showing available capacity | Genuine discrepancy between the Create wizard's quota check and the actual Quotas page — documented as a real Portal limitation rather than forced with a workaround |
| 8 | 7a | Bastion session dropped with "Connection Error" right after starting the domain controller promotion | Expected — promotion always ends in an automatic reboot, which disconnects any active remote session |
| 9 | 7b | Spent several steps in the wrong Defender portal pages before finding sensor settings | Correct path is **System → Settings → Identities**, not the Identities menu's own "Tools" item |
| 10 | 7b | Defender for Identity workspace failed to provision the first time ("MDI instance could not be created") | Verified license (EMS E5) and role (active Global Administrator) were correct — this was a temporary Microsoft-side delay; retried and it succeeded |
| 11 | 7b | The newer "Activate servers" sensor method never listed the server | It depends on Defender for Endpoint device inventory, which the VM wasn't onboarded to — used the classic downloadable installer instead |
| 12 | 7b | Direct blob URL download inside the VM failed with `PublicAccessNotPermitted` | Storage account correctly blocks public access by default — generated a time-limited SAS token instead |
| 13 | 7b | Sensor showed "Not healthy" with 2 open issues right after install | Normal for a first install (NTLM Auditing, Advanced Auditing not yet enabled) — logged as a follow-up Group Policy task rather than blocking completion |

---

## Full key learnings list (all tasks)

- A VM deployed into a subnet from an earlier project **inherits that subnet's NSG** — always check existing rules before assuming a new port is open.
- Azure Bastion traffic to a VM comes from the **AzureBastionSubnet's own IP range**, not the user's browser IP.
- Regional vCPU quota is **shared across every VM** of the same series in a subscription.
- The Azure Portal's live quota checks are not always in sync with the actual Quotas page — the Quotas page is the more trustworthy source.
- A newly attached data disk shows up as Unallocated/Not Initialized — it must be initialized and formatted before Windows can use it.
- Custom Script Extension requires the script to be hosted somewhere Azure can fetch it from (Blob Storage), not uploaded directly from a local computer.
- Installing the AD DS **role** and **promoting** the server to a domain controller are two separate steps — the role alone does nothing until promotion runs.
- A Defender for Identity workspace is provisioned **lazily** on first access, not automatically the instant a license is assigned.
- The newer "Activate servers" sensor flow silently depends on Defender for Endpoint device inventory; the classic manual installer has no such dependency.
- Secure-by-default storage (no public blob access) applies even for internal lab file transfers — SAS tokens are the correct way to share a private blob temporarily.
- "Stopped (deallocated)" is different from just shutting down the OS — only deallocation actually stops billing, while disks/config are preserved.

---

## Infrastructure reused from earlier projects

- **VNet `vnet-lab-sc200` / `subnet-app`** (Project 6) hosts both VMs in this project.
- **Storage account `strakibullab01` / `mycontainer`** (Project 5) was reused twice: once to host the Custom Script Extension script (Task 5), and again to transfer the Defender for Identity sensor installer into the VM (Task 7b).

---

## What's next (beyond this project)

- (Optional follow-up) Configure NTLM Auditing and Advanced Audit Policy via Group Policy so the Defender for Identity sensor reaches a fully "Healthy" status.
- Generate real AD activity (new users, logins) so Defender for Identity has data to analyze, feeding into later SC-200 detection/investigation projects.
- Availability Set/Zone: revisit once subscription quota allows a clean demo.
- Both VMs are currently stopped/deallocated — start them again when the next project needs them.
