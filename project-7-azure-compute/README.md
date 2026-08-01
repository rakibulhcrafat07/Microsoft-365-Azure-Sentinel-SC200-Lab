# Project 7 — Azure Compute: Virtual Machines, AD DS & Defender for Identity

**Goal:** create a Windows and a Linux VM, practice the everyday AZ-104 VM operations (resize, disks, snapshots, extensions, availability), and turn the Windows VM into a real Active Directory Domain Controller so it can generate identity data for a Microsoft Defender for Identity sensor.

**Certification focus:** AZ-104 / SC-200

---

## Why this is a long post

Almost nothing in this project worked on the first try. A VM size that doesn't exist in this region, an NSG that silently blocks RDP, Bastion blocked for a completely different reason than the RDP client was, a resize that hits a shared regional quota, a Portal quota checker that flatly disagrees with the real Quotas page, a Defender for Identity workspace that refuses to provision the first time, and a storage account correctly refusing a public download mid-sensor-install. None of it was hidden — the dead ends are the part worth reading.

AWS EC2 was left out on purpose — this lab stays Azure/Microsoft-only, matching every earlier project.

---

## What was built

| # | Item | Result |
|---|---|---|
| 1 | Windows Server 2022 VM | `vm-win-addc`, reachable via Bastion after 3 rounds of access troubleshooting |
| 2 | Ubuntu VM | `vm-ubuntu-lab`, no public IP, SSH via Bastion |
| 3 | VM resize | `D2s_v7` → `D2ds_v7` on the Linux VM, after a real regional vCPU quota block |
| 4 | Managed disk + snapshot | 32 GiB data disk on `vm-win-addc`, initialized as `D:`, incremental snapshot taken |
| 5 | Custom Script Extension | Installed and verified via a timestamped proof file inside the VM |
| 6 | Availability Zone | Attempted on a demo VM — blocked by a genuine Portal quota-caching bug, documented as-is |
| 7 | AD DS promotion | `vm-win-addc` promoted to the first Domain Controller of a new forest, `rakibul.local` |
| 8 | Defender for Identity sensor | Installed and reporting from `rakibul.local`; 2 minor health issues logged as a follow-up |
| 9 | VM shutdown | Both VMs stopped (deallocated) to end the project cleanly |

All screenshots below live in `images/`, numbered `001`–`143` in the order the work was done.

---

## 7.1 — Windows Server VM setup

**Goal:** Create a Windows Server 2022 VM in the existing lab environment, and prove it's actually reachable — not just "created" in the portal, but something you can log into.

---

### Steps

Starting point: the Virtual Machines list is empty. Nothing has been deployed yet in this project.

[![Starting point](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/001_no_vms_yet.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/001_no_vms_yet.png)

Opened "Create a virtual machine." By default Azure suggests an Ubuntu Linux image — this needed to be changed to Windows.

[![Opened "Create a virtual machine](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/002_create_vm_basics_blank.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/002_create_vm_basics_blank.png)

Searched the Marketplace for "windows server 2022" to find the official Microsoft image (not a third-party one).

[![Searched the Marketplace for "windows server 2022" to find the officia](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/003_image_selector_open.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/003_image_selector_open.png)
[![Searched the Marketplace for "windows server 2022" to find the officia (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/004_marketplace_search_results.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/004_marketplace_search_results.png)

Multiple editions showed up. Picked **Windows Server 2022 Datacenter: Azure Edition — x64 Gen 2**, which is the full GUI version (not the lightweight "Core" edition, since AD DS setup later needs the graphical interface).

[![Multiple editions showed up](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/005_windows_server_2022_editions.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/005_windows_server_2022_editions.png)

Filled in the resource group, VM name (`vm-win-addc`), and other Basics fields. The default size `Standard_B1s` showed a red error: **this size isn't available in East US for this subscription.**

[![Filled in the resource group, VM name (vm-win-addc), and other Basics ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/006_basics_filled_size_error.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/006_basics_filled_size_error.png)

Searched for an available size instead of guessing. Landed on `Standard_D2s_v7` (2 vCPUs, 8 GiB RAM) — confirmed available for this subscription and region.

[![Searched for an available size instead of guessing](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/007_size_dropdown_no_b2s.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/007_size_dropdown_no_b2s.png)
[![Size search box](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/008_select_vm_size_search.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/008_select_vm_size_search.png)
[![D2s_v7 found and available](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/009_d2s_v7_found.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/009_d2s_v7_found.png)

Basics tab fully filled: resource group `RG-Phase1-Lab`, VM name `vm-win-addc`, size `Standard_D2s_v7`, admin username `azureadmin`, a strong password, and inbound port RDP (3389) allowed.

[![Basics tab fully filled](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/010_basics_tab_complete.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/010_basics_tab_complete.png)

Reviewed everything before creating. Networking tab confirmed the VM would join the VNet/subnet already built in Project 6 (`vnet-lab-sc200/subnet-app`), reusing existing infrastructure instead of creating a duplicate.

[![Reviewed everything before creating](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/011_review_create_summary.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/011_review_create_summary.png)
[![Reviewed everything before creating (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/012_networking_tab_summary.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/012_networking_tab_summary.png)

Deployment succeeded. VM, network interface, and public IP were all created with status "OK".

[![Deployment succeeded](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/013_deployment_complete.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/013_deployment_complete.png)

VM confirmed Running, with its public IP (`20.51.164.218`) and private IP (`10.10.1.5`) visible.

[![VM confirmed Running, with its public IP (20](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/014_vm_overview_running.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/014_vm_overview_running.png)

---

### Trying to connect (this is where it got interesting)

Downloaded the native RDP file from the Portal and opened it. Windows showed the normal "unknown publisher" security warnings — clicked through them as expected.

[![Connect menu options](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/015_connect_menu_options.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/015_connect_menu_options.png)
[![Native RDP download page](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/016_native_rdp_download_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/016_native_rdp_download_page.png)
[![RDP file downloaded](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/017_rdp_file_downloaded.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/017_rdp_file_downloaded.png)
[![RDP open warning](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/018_rdp_open_warning.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/018_rdp_open_warning.png)
[![RDP security warning](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/019_rdp_security_warning.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/019_rdp_security_warning.png)

RDP failed anyway: *"Remote Desktop can't connect to the remote computer."*

[![RDP failed anyway](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/020_rdp_connection_failed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/020_rdp_connection_failed.png)

> **Why:** This VM was placed into the same subnet (`subnet-app`) used in Project 6, and that subnet already had a Network Security Group (`nsg-app-subnet`) attached — one that only allowed inbound SSH (port 22), not RDP (port 3389).

Confirmed the NSG only had an SSH rule.

[![Confirmed the NSG only had an SSH rule](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/021_subnet_nsg_shown.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/021_subnet_nsg_shown.png)
[![Confirmed the NSG only had an SSH rule (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/022_nsg_inbound_rules_ssh_only.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/022_nsg_inbound_rules_ssh_only.png)

Added a new inbound rule allowing port 3389 from "My IP."

[![Added a new inbound rule allowing port 3389 from "My IP](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/023_add_rdp_rule_form.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/023_add_rdp_rule_form.png)
[![Added a new inbound rule allowing port 3389 from "My IP (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/024_rdp_rule_created.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/024_rdp_rule_created.png)

Tried RDP again. Still failed, with a different underlying cause this time.

[![Tried RDP again](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/025_rdp_retry_security_warning.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/025_rdp_retry_security_warning.png)
[![Tried RDP again (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/026_rdp_still_failed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/026_rdp_still_failed.png)

Used Azure's built-in "Check access" diagnostic on the Connect page. It reported the port **was** reachable from the source IP — meaning the NSG rule was working, but something else was blocking the connection.

[![Used Azure's built-in "Check access" diagnostic on the Connect page](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/027_vm_overview_connect_menu.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/027_vm_overview_connect_menu.png)
[![Used Azure's built-in "Check access" diagnostic on the Connect page (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/028_check_access_passed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/028_check_access_passed.png)

Restarted the VM in case the RDP listener hadn't fully initialized after the first boot.

[![Restarted the VM in case the RDP listener hadn't fully initialized aft](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/029_restart_vm_confirm.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/029_restart_vm_confirm.png)
[![Restarted the VM in case the RDP listener hadn't fully initialized aft (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/030_restart_success.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/030_restart_success.png)

Still failed, now with a specific error code `0x204`, which usually means the connection is being blocked before it even reaches the VM — most likely the local ISP or network blocking outbound RDP traffic.

[![Still failed, now with a specific error code 0x204, which usually mean](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/031_rdp_error_0x204.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/031_rdp_error_0x204.png)

Switched strategy entirely: deployed Azure Bastion (browser-based access, no RDP client and no port 3389 needed on the local side). Bastion had been deleted at the end of Project 6 to save cost, so it needed to be redeployed here.

[![Switched strategy entirely](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/032_bastion_deploy_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/032_bastion_deploy_page.png)

First Bastion attempt also failed: *"The target machine encountered an error and has closed the connection."*

[![First Bastion attempt also failed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/033_bastion_connect_form.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/033_bastion_connect_form.png)
[![First Bastion attempt also failed (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/034_bastion_connection_error.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/034_bastion_connection_error.png)

> **Why:** Bastion reaches the VM from inside its own subnet (`AzureBastionSubnet`, `10.10.3.0/26`) — not from the user's own IP. The NSG rule from step 24 only allowed RDP from "My IP," so Bastion's traffic was still being blocked.

Added a second inbound rule allowing port 3389 specifically from the Bastion subnet range.

[![Added a second inbound rule allowing port 3389 specifically from the B](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/035_add_bastion_rdp_rule.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/035_add_bastion_rdp_rule.png)
[![Added a second inbound rule allowing port 3389 specifically from the B (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/036_bastion_rule_created.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/036_bastion_rule_created.png)

One more failed attempt (Bastion sometimes needs a minute to fully "warm up" right after deployment).

[![One more failed attempt (Bastion sometimes needs a minute to fully "wa](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/037_bastion_still_failed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/037_bastion_still_failed.png)

Success. The Windows Server 2022 desktop loaded in the browser via Bastion, confirming the VM is fully set up and reachable.

[![Success](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/038_windows_desktop_success.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/038_windows_desktop_success.png)


---

## 7.2 — Linux (Ubuntu) VM setup

**Goal:** Create an Ubuntu VM and practice SSH-based access, to contrast against the Windows RDP experience in Task 1.

---

### Steps

Created a new VM `vm-ubuntu-lab` using image **Ubuntu Server 24.04 LTS**, size `Standard_D2s_v7` (same verified-working size from Task 1), password authentication, and inbound port SSH (22) allowed.

[![Created a new VM vm-ubuntu-lab using image Ubuntu Server 24](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/039_basics_filled.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/039_basics_filled.png)

On the Networking tab, joined the same existing VNet/subnet (`vnet-lab-sc200/subnet-app`) as the Windows VM, but this time set **Public IP: None**. Since Bastion was already being used successfully for the Windows VM, there was no need to expose this VM to the public internet at all — a more secure setup than Task 1's public-IP approach.

[![On the Networking tab, joined the same existing VNet/subnet (vnet-lab-](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/040_networking_no_public_ip.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/040_networking_no_public_ip.png)

Deployment succeeded. Only a network interface was created (no public IP resource), confirming the no-public-IP choice took effect.

[![Deployment succeeded](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/041_deployment_complete.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/041_deployment_complete.png)

Connected via Bastion using SSH protocol instead of RDP. Landed directly at the `azureadmin@vm-ubuntu-lab:~$` prompt — no NSG troubleshooting needed this time, because Bastion's SSH path used a different mechanism than the RDP rule that had to be added for Windows.

[![Connected via Bastion using SSH protocol instead of RDP](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/042_bastion_ssh_success.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/042_bastion_ssh_success.png)


---

## 7.3 — VM resize

**Goal:** Change a running VM's size to a bigger one, and observe what actually happens during a resize (including regional quota limits — a real AZ-104 exam topic).

Resize was performed on the **Linux VM** (`vm-ubuntu-lab`) rather than the Windows VM, on purpose — the Windows VM is planned to become an Active Directory Domain Controller later in this project, so it's better to leave it untouched once its role is more critical.

---

### Steps

Linux VM confirmed Running at its current size, `Standard_D2s_v7`.

[![Linux VM confirmed Running at its current size, Standard_D2s_v7](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/043_vm_overview_before_resize.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/043_vm_overview_before_resize.png)

Opened the VM's Size page (Availability + scale → Size). A note here says resizing a running VM causes it to restart.

[![Opened the VM's Size page (Availability + scale → Size)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/044_size_page_current_d2s_v7.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/044_size_page_current_d2s_v7.png)

Selected `Standard_D4s_v7` (4 vCPUs, 16 GiB — double the current size) and confirmed the resize.

[![Selected Standard_D4s_v7 (4 vCPUs, 16 GiB — double the current size) a](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/045_resize_confirm_d4s_v7.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/045_resize_confirm_d4s_v7.png)

Resize failed: *"exceeding the approved Total Regional Cores quota... Current Limit: 4, Current Usage: 4."* The subscription's D-series v7 family is capped at 4 vCPUs total in East US, and both VMs (2 vCPUs each) were already using all of it.

[![Resize failed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/046_resize_failed_quota.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/046_resize_failed_quota.png)

Stopped (deallocated) the Windows VM to free up quota for the resize test.

[![Stopped (deallocated) the Windows VM to free up quota for the resize t](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/047_stop_windows_vm_dialog.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/047_stop_windows_vm_dialog.png)
[![Stopped (deallocated) the Windows VM to free up quota for the resize t (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/048_bastion_disconnected_after_stop.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/048_bastion_disconnected_after_stop.png)

Tried resizing again — same error, even though the Windows VM was now stopped. The Portal's quota counter had not refreshed yet (a caching delay, not a real quota problem).

[![Tried resizing again — same error, even though the Windows VM was now ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/049_resize_failed_again_stale_quota.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/049_resize_failed_again_stale_quota.png)

Tried a different VM family (`D4s_v5`) as an alternative. This family showed **"Request quota"** — meaning it isn't approved for this subscription at all, unrelated to the current usage issue.

[![Tried a different VM family (D4s_v5) as an alternative](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/050_d4s_v5_not_available.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/050_d4s_v5_not_available.png)

Checked the "Insufficient quota" categories on the Size page to see exactly which sizes were blocked and why.

[![Checked the "Insufficient quota" categories on the Size page to see ex](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/051_insufficient_quota_categories.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/051_insufficient_quota_categories.png)

Refreshed the page and confirmed the current size was still `Standard_D2s_v7`.

[![Refreshed the page and confirmed the current size was still Standard_D](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/052_size_page_refreshed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/052_size_page_refreshed.png)

Instead of chasing the D4s_v7 quota issue further, picked `Standard_D2ds_v7` — same 2 vCPU / 8 GiB tier as before, but a different disk-optimized variant, which was not blocked by the quota. Resize started successfully.

[![Instead of chasing the D4s_v7 quota issue further, picked Standard_D2d](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/053_resizing_to_d2ds_v7.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/053_resizing_to_d2ds_v7.png)

Watched the resize happen live through the Bastion terminal session: the VM itself printed *"The system will power off now!"* — direct proof that resizing triggers a reboot, exactly as the earlier warning said.

[![Watched the resize happen live through the Bastion terminal session](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/054_vm_powering_off_for_resize.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/054_vm_powering_off_for_resize.png)

Reconnected via Bastion after the resize. The login banner showed a fresh session (`Last login... from 10.10.3.4`, the Bastion subnet's IP), confirming the VM came back up successfully at its new size.

[![Reconnected via Bastion after the resize](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/055_reconnected_after_resize.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/055_reconnected_after_resize.png)

---

### Outcome

The resize from `D2s_v7` → `D2ds_v7` succeeded and demonstrated the resize mechanics (restart-on-resize) clearly. The larger upsize to `D4s_v7` was blocked by a real regional quota limit shared across every VM in the subscription — a genuine constraint of running on a free-trial subscription, not a configuration mistake.


---

## 7.4 — Managed disk & snapshot

**Goal:** Attach an extra data disk to the Windows VM, bring it online inside Windows, and take a snapshot (a point-in-time backup) of that disk.

Done on the **Windows VM** (`vm-win-addc`) on purpose — in a real Active Directory setup, the AD database, logs, and SYSVOL folder are always kept on a separate data drive rather than the OS drive. This disk will become that data drive once AD DS is configured later in this project.

---

### Steps

The Windows VM had been stopped earlier (Task 3) to free up quota, so it needed to be started again before this task.

[![The Windows VM had been stopped earlier (Task 3) to free up quota, so ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/056_bastion_disconnected_vm_stopped.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/056_bastion_disconnected_vm_stopped.png)

On the first connection attempt after starting the VM, the Portal showed a warning that the VM agent wasn't ready yet — the VM needs a short warm-up period right after starting.

[![On the first connection attempt after starting the VM, the Portal show](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/057_vm_agent_not_ready_after_start.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/057_vm_agent_not_ready_after_start.png)

Second attempt succeeded — back on the Windows desktop via Bastion.

[![Second attempt succeeded — back on the Windows desktop via Bastion](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/058_windows_desktop_reconnected.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/058_windows_desktop_reconnected.png)

Confirmed both VMs (Windows and Linux) were Running before continuing.

[![Confirmed both VMs (Windows and Linux) were Running before continuing](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/059_vm_list_both_running.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/059_vm_list_both_running.png)

Opened the Windows VM's Disks page. Only the OS disk existed; "Data disks: 0."

[![Opened the Windows VM's Disks page](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/060_disks_page_empty.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/060_disks_page_empty.png)

Clicked "Create and attach a new disk" and filled in: name `vm-win-addc-datadisk01`, storage type **Standard SSD** (cheaper than Premium, fine for a lab), size **32 GiB**.

[![Clicked "Create and attach a new disk" and filled in](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/061_new_disk_form_filled.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/061_new_disk_form_filled.png)

Clicked Apply. Two success notifications appeared: "Successfully created disk" and "Updated virtual machine" — the disk now shows in the Data disks list at LUN 0.

[![Clicked Apply](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/062_disk_created_attached.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/062_disk_created_attached.png)

Opened Disk Management inside Windows (`diskmgmt.msc`). Azure automatically prompted to **Initialize Disk** for the new 32 GB disk, which showed as "Not Initialized." Chose **MBR** partition style (simpler, and more than enough for a 32 GB disk — GPT is only needed for disks over 2 TB or special boot scenarios).

[![Opened Disk Management inside Windows (diskmgmt](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/063_initialize_disk_dialog.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/063_initialize_disk_dialog.png)

After initializing and creating a New Simple Volume (formatted NTFS, drive letter **D:**, labeled "Disk Addition"), Disk Management showed **Disk 1: 32.00 GB, Online, Healthy (Primary Partition)**.

[![After initializing and creating a New Simple Volume (formatted NTFS, d](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/064_disk_management_online.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/064_disk_management_online.png)

Confirmed in File Explorer: "Disk Addition (D:)" now appears as a normal, usable drive with 31.9 GB free.

[![Confirmed in File Explorer](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/065_file_explorer_d_drive.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/065_file_explorer_d_drive.png)

---

### Snapshot

Opened the disk's own resource page in the Portal (not the VM's Disks tab, but the Disk resource itself) and clicked "Create snapshot."

[![Opened the disk's own resource page in the Portal (not the VM's Disks ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/066_disk_resource_overview.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/066_disk_resource_overview.png)

The snapshot form auto-filled the source disk. Kept the defaults: **Incremental** snapshot type (only stores the difference from the last snapshot, cheaper) and **Standard HDD** storage (cheapest, fine for a backup that isn't accessed often). Named it `vm-win-addc-datadisk01-snapshot01`.

[![The snapshot form auto-filled the source disk](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/067_create_snapshot_form.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/067_create_snapshot_form.png)

Snapshot deployment succeeded, confirmed with status "OK" in the deployment details.

[![Snapshot deployment succeeded, confirmed with status "OK" in the deplo](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/068_snapshot_deployment_complete.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/068_snapshot_deployment_complete.png)
[![Snapshot deployment succeeded, confirmed with status "OK" in the deplo (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/069_snapshot_deployment_details.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/069_snapshot_deployment_details.png)

---

### Outcome

The data disk is attached, initialized, formatted, and usable inside Windows as drive D:. A snapshot of that disk now exists as a recoverable backup point — both pieces of the task completed and verified.


---

## 7.5 — VM extension (custom script)

**Goal:** Install a VM extension (a small agent that runs automation inside the VM after deployment) and prove that it actually executed — not just that Azure reported success.

Used the **Custom Script Extension**, which downloads and runs a PowerShell script inside the VM. Installed on the Windows VM (`vm-win-addc`).

---

### Steps

Opened the VM's "Extensions + applications" page. Nothing installed yet ("No resource extensions found").

[![Opened the VM's "Extensions + applications" page](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/070_extensions_page_empty.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/070_extensions_page_empty.png)

Clicked "+ Add" to browse the extension gallery — dozens of options are available (monitoring agents, security tools, automation agents, etc.).

[![Clicked "+ Add" to browse the extension gallery — dozens of options ar](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/071_install_extension_gallery.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/071_install_extension_gallery.png)

Selected **Custom script extension** (published by Microsoft Corp.). Its description: it can download PowerShell scripts and files from Azure storage and run them on the VM, useful for post-deployment automation.

[![Selected Custom script extension (published by Microsoft Corp](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/072_custom_script_extension_card.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/072_custom_script_extension_card.png)
[![Selected Custom script extension (published by Microsoft Corp (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/073_custom_script_extension_selected.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/073_custom_script_extension_selected.png)

The configuration screen asks for a script file. Note: "Browse" here does **not** browse the local computer — it browses an Azure Storage Account, since the extension needs to fetch the script from Azure, not from a local file.

[![The configuration screen asks for a script file](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/074_configure_extension_form.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/074_configure_extension_form.png)

Wrote a simple test script locally, saved as `install-extension-test.ps1` (had to choose "All Files" when saving in Notepad so it wouldn't default to `.txt`). The script creates a folder and a text file with a timestamp — a simple, verifiable proof that the script actually ran.

[![Wrote a simple test script locally, saved as install-extension-test](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/075_powershell_script_file_icon.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/075_powershell_script_file_icon.png)
[![Wrote a simple test script locally, saved as install-extension-test (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/076_powershell_script_content.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/076_powershell_script_content.png)

```powershell
New-Item -Path "C:\ExtensionTest" -ItemType Directory -Force
Set-Content -Path "C:\ExtensionTest\extension-proof.txt" -Value "Custom Script Extension ran successfully on $(Get-Date)"
```

Browsed to the storage account already built in Project 5 (`strakibullab01`) and its existing `mycontainer` blob container, reusing infrastructure from an earlier project instead of creating a new one.

[![Browsed to the storage account already built in Project 5 (strakibulla](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/077_storage_accounts_browse.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/077_storage_accounts_browse.png)
[![Browsed to the storage account already built in Project 5 (strakibulla (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/078_storage_containers_list.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/078_storage_containers_list.png)

Uploaded `install-extension-test.ps1` into the container. It now sits alongside `Test.txt` from Project 5.

[![Uploaded install-extension-test](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/079_script_uploaded_to_blob.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/079_script_uploaded_to_blob.png)

Reviewed the extension configuration (script file path filled in automatically after selecting the uploaded blob) and clicked Create.

[![Reviewed the extension configuration (script file path filled in autom](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/080_review_create_extension.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/080_review_create_extension.png)

Deployment succeeded.

[![Deployment succeeded](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/081_extension_deployment_succeeded.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/081_extension_deployment_succeeded.png)

---

### Verifying it actually ran

Connected to the VM via Bastion and opened `C:\ExtensionTest` in File Explorer. The file `extension-proof.txt` was there, dated at the exact time of deployment.

[![Connected to the VM via Bastion and opened C](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/082_extension_test_folder_created.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/082_extension_test_folder_created.png)

Opened the file in Notepad. It read: *"Custom Script Extension ran successfully on 07/31/2026 13:17:02"* — solid proof the script executed inside the VM, not just that Azure reported "success" in the Portal.

[![Opened the file in Notepad](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/083_extension_proof_file_content.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/083_extension_proof_file_content.png)


---

## 7.6 — Availability zone (attempted)

**Goal:** Deploy a VM pinned to a specific Availability Zone, to understand how zone placement works in Azure.

Availability Zone has to be chosen **at VM creation time** — it can't be added to an already-running VM afterward. Since both existing VMs (Windows and Linux) were already using the full regional vCPU quota, a brand-new demo VM (`vm-az-demo`) was needed to test this feature.

---

### Steps

Started creating `vm-az-demo` with **Availability options: Availability zone**, **Zone 1** selected. The Size field showed a red error: *"2 vCPUs are needed for this configuration, but only 0 vCPUs (of 4) remain in your subscription."*

[![Started creating vm-az-demo with Availability options](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/084_demo_vm_basics_quota_error.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/084_demo_vm_basics_quota_error.png)

Stopped (deallocated) the Linux VM to free up 2 vCPUs of quota, expecting the new VM to then have room.

[![Stopped (deallocated) the Linux VM to free up 2 vCPUs of quota, expect](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/085_linux_vm_stopped_to_free_quota.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/085_linux_vm_stopped_to_free_quota.png)

Tried again (including refreshing the quota and starting the wizard fresh) — same "0 of 4 remain" error persisted.

[![Tried again (including refreshing the quota and starting the wizard fr](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/086_quota_error_still_showing.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/086_quota_error_still_showing.png)

Checked the actual subscription-level Quotas page (Usage + quotas → Standard Dsv7 Family vCPUs, East US). It showed **"2 of 4"** in use — meaning 2 vCPUs should have been free, directly contradicting what the VM Create wizard was reporting.

[![Checked the actual subscription-level Quotas page (Usage + quotas → St](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/087_quotas_page_actual_usage_2of4.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/087_quotas_page_actual_usage_2of4.png)

---

### Outcome

This is a genuine discrepancy between the Portal's VM Create wizard (which checks quota through one code path) and the actual Quotas service (a different, more authoritative source). Rather than force a workaround — for example, deploying via Azure CLI to bypass the wizard's stale check — this was documented honestly as a real tooling limitation encountered while working within a free-trial subscription.

The Availability Zone **concept** was still explored through the Basics tab during this attempt: the "Zone options" setting offers a choice between **Self-selected zone** (pick Zone 1, 2, or 3 yourself) and **Azure-selected zone** (let Azure pick the best zone automatically) — both options were visible and understood, even though the deployment itself couldn't complete due to the quota issue above.


---

## 7.7 — Active Directory Domain Services (AD DS)

**Goal:** Install the Active Directory Domain Services role on the Windows VM and promote it to a fully functioning Domain Controller. This generates the real identity data (users, logins, group changes) that a Microsoft Defender for Identity sensor needs to analyze — the main reason this VM exists in the project.

---

### Steps

Opened Server Manager on the Windows VM (opens automatically, or from the taskbar). Closed an informational popup suggesting Windows Admin Center/Azure Arc as alternatives — Server Manager was used instead since it directly matches the AZ-104 exam objectives.

[![Opened Server Manager on the Windows VM (opens automatically, or from ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/088_server_manager_dashboard.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/088_server_manager_dashboard.png)

From Server Manager's "Manage" menu, opened **Add Roles and Features Wizard**. The "Before You Begin" page confirmed the destination server as `vm-win-addc`.

[![From Server Manager's "Manage" menu, opened Add Roles and Features Wiz](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/089_add_roles_wizard_start.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/089_add_roles_wizard_start.png)

Selected **Role-based or feature-based installation** (the standard option for configuring a single server, as opposed to Remote Desktop Services installation).

[![Selected Role-based or feature-based installation (the standard option](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/090_installation_type.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/090_installation_type.png)

Confirmed `vm-win-addc` (10.10.1.5) as the target server from the server pool.

[![Confirmed vm-win-addc (10](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/091_server_selection.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/091_server_selection.png)

Checked **Active Directory Domain Services** in the roles list. The description on the right explained: AD DS stores information about network objects (users, computers) and lets administrators manage that information and control access across the network.

[![Checked Active Directory Domain Services in the roles list](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/092_server_roles_ad_ds_checked.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/092_server_roles_ad_ds_checked.png)

On the Features page, **Group Policy Management** was already automatically checked — it's a required companion tool for AD DS, added automatically as a dependency. Nothing else needed to be selected.

[![On the Features page, Group Policy Management was already automaticall](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/093_features_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/093_features_page.png)

An informational page about AD DS appeared, noting that production environments should have at least two domain controllers for redundancy (not required for this single-server lab).

[![An informational page about AD DS appeared, noting that production env](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/094_ad_ds_info_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/094_ad_ds_info_page.png)

Reviewed the final list of what would be installed: Active Directory Domain Services, Group Policy Management, Remote Server Administration Tools, AD DS and AD LDS Tools, Active Directory module for PowerShell, AD DS Tools, Active Directory Administrative Center, and AD DS Snap-Ins.

[![Reviewed the final list of what would be installed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/095_confirmation_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/095_confirmation_page.png)

Installation completed: **"Configuration required. Installation succeeded on vm-win-addc."** A link appeared: **"Promote this server to a domain controller"** — this is the next required step, since installing the role alone does not make the server a functioning Domain Controller yet.

[![Installation completed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/096_installation_progress_succeeded.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/096_installation_progress_succeeded.png)

Back on the Server Manager Dashboard, a new **AD DS** section now appears in "Roles and Server Groups" (Roles count went from 1 to 2), confirming the role is present on the server.

[![Back on the Server Manager Dashboard, a new AD DS section now appears ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/097_server_manager_ad_ds_added.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/097_server_manager_ad_ds_added.png)

---

### Promoting the server to a Domain Controller

Returned to the Server Manager Dashboard notification flag and clicked **"Promote this server to a domain controller"** to launch the AD DS Configuration Wizard.

[![Returned to the Server Manager Dashboard notification flag and clicked](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/098_promote_to_dc_notification.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/098_promote_to_dc_notification.png)

The wizard's **Deployment Configuration** screen defaulted to "Add a domain controller to an existing domain" — the wrong option, since no domain exists yet on this server.

[![The wizard's Deployment Configuration screen defaulted to "Add a domai](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/099_deployment_config_default_wrong.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/099_deployment_config_default_wrong.png)

Switched to **"Add a new forest"** and entered the root domain name `rakibul.local`. This is the correct choice for the very first domain controller — it creates a brand-new Active Directory forest from scratch.

[![Switched to "Add a new forest" and entered the root domain name rakibu](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/100_deployment_config_new_forest.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/100_deployment_config_new_forest.png)

On **Domain Controller Options**, left the Forest/Domain functional level at the default "Windows Server 2016," kept DNS Server and Global Catalog checked, and set a Directory Services Restore Mode (DSRM) recovery password.

[![On Domain Controller Options, left the Forest/Domain functional level ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/101_domain_controller_options.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/101_domain_controller_options.png)

The **DNS Options** screen showed a yellow warning that a DNS delegation could not be created. This is expected and harmless — it only matters when a domain is meant to be a child of an existing parent DNS zone, which isn't the case for this isolated lab forest.

[![The DNS Options screen showed a yellow warning that a DNS delegation c](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/102_dns_options_warning.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/102_dns_options_warning.png)

The wizard auto-generated the NetBIOS domain name `RAKIBUL` on the **Additional Options** screen — left as-is.

[![The wizard auto-generated the NetBIOS domain name RAKIBUL on the Addit](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/103_additional_options_netbios.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/103_additional_options_netbios.png)

Left the AD DS database, log files, and SYSVOL folder locations at their Windows defaults on the **Paths** screen.

[![Left the AD DS database, log files, and SYSVOL folder locations at the](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/104_paths_default.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/104_paths_default.png)

The **Review Options** screen summarized the whole configuration: new forest, domain `rakibul.local`, NetBIOS `RAKIBUL`, DNS Server and Global Catalog both enabled.

[![The Review Options screen summarized the whole configuration](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/105_review_options_summary.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/105_review_options_summary.png)

**Prerequisites Check** passed with a green "All prerequisite checks passed successfully" message. Two informational (non-blocking) notes appeared about cryptography compatibility settings and static IP recommendations — both normal for a lab VM. Clicked **Install**.

[![Prerequisites Check passed with a green "All prerequisite checks passe](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/106_prerequisites_check_passed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/106_prerequisites_check_passed.png)

Immediately after clicking Install, the Bastion remote session dropped with a "Connection Error" dialog. This is expected — promoting a server to a domain controller always ends with an automatic reboot, which disconnects any active remote session. Waited a few minutes before reconnecting.

[![Immediately after clicking Install, the Bastion remote session dropped](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/107_connection_error_during_reboot.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/107_connection_error_during_reboot.png)

After reconnecting via Bastion, Server Manager's Dashboard now listed **both AD DS and DNS** as active roles (previously only AD DS, without DNS) — the first sign the promotion succeeded.

[![After reconnecting via Bastion, Server Manager's Dashboard now listed ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/108_reconnected_ad_ds_dns_roles.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/108_reconnected_ad_ds_dns_roles.png)

Opened the AD DS section in Server Manager and checked the **Services** panel: **"Active Directory Domain Services" (NTDS)** and **"Kerberos Key Distribution Center" (Kdc)** were both **Running**. The KDC service only runs on an actual, working domain controller — this is the clearest proof the promotion completed successfully.

[![Opened the AD DS section in Server Manager and checked the Services pa](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/109_services_ntds_kdc_running.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/109_services_ntds_kdc_running.png)

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

- Install the **Microsoft Defender for Identity sensor** on this now-functioning Domain Controller
- Generate some real activity (a new AD user, a login) so Defender for Identity has actual data to work with, setting up later SC-200 detection projects


---

## 7.8 — Microsoft Defender for Identity sensor

**Goal:** Install the Microsoft Defender for Identity (MDI) sensor on `vm-win-addc`, now that it's a working Domain Controller, so it can monitor the on-premises Active Directory environment for suspicious activity — the SC-200-relevant reason this VM exists.

---

### Part 1 — Finding the right page in the Defender portal

This part took several wrong turns before landing on the correct settings page — all of them are documented here honestly, since dead ends and how to recover from them are as useful to read as the steps that worked.

Opened `security.microsoft.com`, the unified Microsoft Defender portal, to begin locating Defender for Identity sensor settings.

[![Opened security](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/110_defender_portal_home.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/110_defender_portal_home.png)

Expanded the **Identities** menu in the left sidebar to see its sub-pages: Dashboard, Coverage & Maturity, Service accounts, Password protection, Tools.

[![Expanded the Identities menu in the left sidebar to see its sub-pages](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/111_identities_menu_expanded.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/111_identities_menu_expanded.png)

Clicked "Tools" under Identities — this actually opened a generic "Welcome to Microsoft Defender for Business" onboarding screen, unrelated to sensor management. Wrong turn #1.

[![Clicked "Tools" under Identities — this actually opened a generic "Wel](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/112_wrong_turn_defender_for_business.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/112_wrong_turn_defender_for_business.png)

The *correct* Identities → Tools page (Documentation, Sizing Tool, Readiness Script, PowerShell module) — confirmed MDI licensing was active, but still not the sensor download page.

[![The correct Identities → Tools page (Documentation, Sizing Tool, Readi](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/113_identity_tools_page.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/113_identity_tools_page.png)

Searching "sensors" in the top search bar accidentally led into a **Defender for Business onboarding wizard** ("Let's give people access") — an unrelated setup flow. Wrong turn #2, cancelled.

[![Searching "sensors" in the top search bar accidentally led into a Defe](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/114_wrong_turn_onboarding_wizard.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/114_wrong_turn_onboarding_wizard.png)

Returned to the Defender portal Home page to try a different navigation path: **System → Settings** instead of the Identities menu directly.

[![Returned to the Defender portal Home page to try a different navigatio](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/115_back_to_home.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/115_back_to_home.png)

Found the correct **Settings** hub, listing all setting categories including **"Identities"** ("General settings for identities") — this was the right link all along.

[![Found the correct Settings hub, listing all setting categories includi](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/116_settings_hub_identities_link.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/116_settings_hub_identities_link.png)

---

### Part 2 — First-time workspace provisioning

Clicking "Identities" here triggered a one-time setup screen: **"Hang on! We're preparing your Microsoft Defender for Identity workspace."** This confirmed the MDI workspace is provisioned lazily on first access, not automatically the moment the license is assigned.

[![Clicking "Identities" here triggered a one-time setup screen](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/117_workspace_provisioning_loading.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/117_workspace_provisioning_loading.png)

Provisioning failed with **"Something went wrong — MDI instance could not be created."** Rather than assume misconfiguration, this was checked methodically:

[![Provisioning failed with "Something went wrong — MDI instance could no](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/118_workspace_provisioning_failed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/118_workspace_provisioning_failed.png)

Verified in the Microsoft 365 admin center that the correct license (**Enterprise Mobility + Security E5**, which includes Defender for Identity) was assigned to the signed-in account — ruled out a licensing problem.

[![Verified in the Microsoft 365 admin center that the correct license (E](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/119_license_check_ems_e5.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/119_license_check_ems_e5.png)

Verified in the Entra admin center that the account held an **active** Global Administrator role (not just "eligible," which would have needed PIM activation) — ruled out a permissions problem.

[![Verified in the Entra admin center that the account held an active Glo](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/120_global_admin_role_confirmed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/120_global_admin_role_confirmed.png)

With both licensing and permissions confirmed correct, the failure was treated as a temporary backend delay on Microsoft's side. Retried after a short wait.

On retry, the **Identity Security** page loaded successfully, showing Activation / Sensors / Service Accounts Classification tabs. (The Activation tab's server list briefly still showed "Failed to load data" — a separate, smaller issue, addressed below.)

[![On retry, the Identity Security page loaded successfully, showing Acti](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/121_identity_security_page_loaded.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/121_identity_security_page_loaded.png)

---

### Part 3 — Choosing an install method

The **Sensors** tab loaded correctly, showing **"No sensors installed."** The workspace was now ready, just empty.

[![The Sensors tab loaded correctly, showing "No sensors installed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/122_sensors_tab_none_installed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/122_sensors_tab_none_installed.png)

Clicked **"Add sensor."** A dialog offered two paths: the newer **"Activate servers"** method (works only for servers already onboarded into Defender for Endpoint's device inventory) and the older **"Continue with the classic sensor"** method (a manual downloadable installer).

[![Clicked "Add sensor](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/123_add_sensor_dialog_two_options.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/123_add_sensor_dialog_two_options.png)

Tried the Activation tab's server list again — it kept showing "Failed to load data," which turned out to be because `vm-win-addc` was never onboarded to Defender for Endpoint, so the newer activation method had nothing to list.

[![Tried the Activation tab's server list again — it kept showing "Failed](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/124_activation_tab_failed_to_load.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/124_activation_tab_failed_to_load.png)

Switched to **"Continue with the classic sensor"** instead. This opened a panel with a **"Download installer"** button and a generated **Access key** — everything needed for a manual install, with no Defender for Endpoint dependency.

[![Switched to "Continue with the classic sensor" instead](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/125_classic_sensor_access_key_panel.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/125_classic_sensor_access_key_panel.png)

---

### Part 4 — Getting the installer onto the Domain Controller

Bastion sessions don't support drag-and-drop or clipboard file transfer for actual files, so the same technique from Task 5 (Custom Script Extension) was reused: upload to Blob Storage, then download from inside the VM.

Uploaded the downloaded `Azure ATP Sensor Setup.zip` (84.55 MiB) to the existing Project 5 storage account (`strakibullab01` → `mycontainer`) — reusing infrastructure across projects rather than creating something new.

[![Uploaded the downloaded Azure ATP Sensor Setup](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/126_upload_installer_to_storage.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/126_upload_installer_to_storage.png)

Upload completed; the container now listed three files, including the new sensor zip alongside earlier test files.

[![Upload completed; the container now listed three files, including the ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/127_blob_uploaded_confirmed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/127_blob_uploaded_confirmed.png)

Opened a fresh Bastion connection to `vm-win-addc`. Username/password were already saved from the earlier session.

[![Opened a fresh Bastion connection to vm-win-addc](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/128_bastion_connect_panel.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/128_bastion_connect_panel.png)

Tried downloading the zip directly inside the VM using the plain blob URL — got an **XML error: `PublicAccessNotPermitted`**. This confirmed the storage account correctly blocks anonymous public access by default (the same secure-by-default behavior explored back in the Storage project).

[![Tried downloading the zip directly inside the VM using the plain blob ](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/129_public_access_error_xml.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/129_public_access_error_xml.png)

Generated a **SAS (Shared Access Signature) token** for the blob — Read-only permission, HTTPS-only, same-day expiry — the proper, time-limited way to allow a private download without opening the storage account to the whole internet.

[![Generated a SAS (Shared Access Signature) token for the blob — Read-on](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/130_generate_sas_token.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/130_generate_sas_token.png)

Pasted the SAS URL into the VM's browser. The download succeeded, and the zip was extracted, revealing three items: an `NPCAP` driver folder, the `Azure ATP Sensor Setup` application, and a `SensorInstallationConfiguration.json` file.

[![Pasted the SAS URL into the VM's browser](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/131_sas_download_extracted_in_vm.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/131_sas_download_extracted_in_vm.png)

Confirmed both the downloaded zip and its extracted contents were present and correct before running the installer.

[![Confirmed both the downloaded zip and its extracted contents were pres](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/132_downloads_folder_zip_and_extracted.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/132_downloads_folder_zip_and_extracted.png)
[![Confirmed both the downloaded zip and its extracted contents were pres (2)](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/133_extracted_folder_contents.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/133_extracted_folder_contents.png)

---

### Part 5 — Installing the sensor

Ran the installer. The **Microsoft Defender for Identity** setup wizard launched (English selected).

[![Ran the installer](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/134_installer_wizard_language.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/134_installer_wizard_language.png)

On **Sensor deployment type**, selected **"Sensor"** (installed directly on the domain controller) — the correct choice, since `vm-win-addc` is itself the domain controller, as opposed to "Standalone Sensor" (needs port-mirroring) or the AD FS/Entra Connect option.

[![On Sensor deployment type, selected "Sensor" (installed directly on th](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/135_sensor_deployment_type_selected.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/135_sensor_deployment_type_selected.png)

The **Configure the Sensor** screen appeared with an empty Access key field and the default installation path.

[![The Configure the Sensor screen appeared with an empty Access key fiel](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/136_configure_sensor_empty_key.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/136_configure_sensor_empty_key.png)

Pasted the Access key generated earlier (clipboard sync worked through the Bastion session).

[![Pasted the Access key generated earlier (clipboard sync worked through](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/137_configure_sensor_key_pasted.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/137_configure_sensor_key_pasted.png)

Installation in progress, showing both a per-component progress bar and an overall progress bar.

[![Installation in progress, showing both a per-component progress bar an](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/138_installation_progress.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/138_installation_progress.png)

**"Installation completed successfully"** with a green checkmark.

[!["Installation completed successfully" with a green checkmark](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/139_installation_completed.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/139_installation_completed.png)

---

### Part 6 — Verifying the sensor

Back in the Defender portal's Sensors tab, `vm-win-addc` now appeared as a registered sensor — Type: Domain controller, Domain: `rakibul.local`, Version 2.255.19295.47272 — but with Health status **"Not healthy"** and 2 open issues.

[![Back in the Defender portal's Sensors tab, vm-win-addc now appeared as](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/140_sensor_registered_not_healthy.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/140_sensor_registered_not_healthy.png)

Opened the health issue details: **"NTLM Auditing is not enabled"** and **"Services Advanced Auditing is not enabled,"** both Medium severity. These are common first-install issues that need Group Policy changes (Advanced Audit Policy Configuration) to resolve — not something the installer configures automatically. Documented here as an honest finding rather than fixed immediately, since the core goal (sensor installed and reporting to the portal) was achieved.

[![Opened the health issue details](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/141_health_issues_detail.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/141_health_issues_detail.png)

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

## 7.9 — VM stop / deallocate

**Goal:** Shut down both lab virtual machines properly at the end of the project so they stop consuming subscription credit and regional vCPU quota while not in active use.

In Azure, simply shutting down a VM's operating system from inside Windows/Linux is not enough to stop billing — the VM has to be **Stopped (Deallocated)** from the Azure control plane itself, which releases the underlying compute hardware.

---

### Steps

Selected `vm-win-addc` in the Azure Portal and clicked **Stop**. The VM's Status changed to **"Stopped (deallocated)"**, and a "Successfully stopped virtual machine" notification confirmed it.

[![Selected vm-win-addc in the Azure Portal and clicked Stop](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/142_vm_win_addc_stopped.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/142_vm_win_addc_stopped.png)

Repeated the same action for `vm-ubuntu-lab`. Its Status also shows **"Stopped (deallocated)"**, with a matching success notification.

[![Repeated the same action for vm-ubuntu-lab](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/raw/main/project-7-azure-compute/images/143_vm_ubuntu_lab_stopped.png)](/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/project-7-azure-compute/images/143_vm_ubuntu_lab_stopped.png)

Both VMs retain their disks, configuration, and network settings while stopped — they can be started again later without losing any of the work done in this project (including the AD DS forest and the installed Defender for Identity sensor).

---

### Key learnings

- "Stopped (deallocated)" is different from just shutting down the OS — only deallocation actually releases the compute resources and stops per-hour billing.
- Stopping a VM does not delete anything — disks, NICs, and configuration are preserved, so the domain controller and sensor setup will still be there the next time the VM is started.


---

## Mistakes & fixes (quick reference)

| Mistake | Fix |
|---|---|
| `Standard_B1s` unavailable for the Windows VM in East US | Switched to `Standard_D2s_v7` |
| RDP failed after VM creation | Inherited subnet's NSG only allowed SSH — added a port 3389 rule from "My IP" |
| RDP still failed (error `0x204`) | Local ISP was blocking outbound RDP entirely — switched to Azure Bastion |
| Bastion failed too, on the first try | Bastion traffic comes from `AzureBastionSubnet`, not "My IP" — added a second NSG rule for that range |
| Resize to `D4s_v7` blocked | Regional vCPU quota (4) was already fully used by both VMs — freed quota, then used `D2ds_v7` instead |
| Resize kept failing even after freeing quota | Portal's quota check was stale/cached — waited and picked a size within the confirmed real quota |
| Zone-pinned demo VM couldn't deploy despite quota showing room | Genuine mismatch between the Create wizard's quota check and the real Quotas page — documented as a real Portal limitation |
| Bastion session dropped mid-AD-DS-promotion | Expected — promotion always ends in an automatic reboot |
| Several wrong turns finding the Defender for Identity sensor page | Correct path is **System → Settings → Identities**, not the Identities menu's own "Tools" item |
| MDI workspace failed to provision the first time | License and role were both correct — a temporary Microsoft-side delay; retried and it worked |
| New "Activate servers" sensor method never listed the server | It needs Defender for Endpoint device inventory, which this VM wasn't onboarded to — used the classic installer instead |
| Direct blob download inside the VM failed (`PublicAccessNotPermitted`) | Storage account correctly blocks public access — generated a SAS token instead |
| Sensor showed "Not healthy" right after install | Normal for a first install — NTLM/Advanced Auditing need Group Policy, logged as a follow-up |

---

## Key learnings

1. **A VM inherits its subnet's NSG** — always check existing rules from an earlier project before assuming a new port is open.
2. **Bastion traffic comes from its own subnet**, not the user's IP — NSG rules for "My IP" don't cover it automatically.
3. **Regional vCPU quota is shared** across every VM of the same series in a subscription.
4. **The Portal's live quota checks can lag behind the real Quotas page** — when they disagree, trust the Quotas page.
5. **A new data disk needs initializing and formatting** before Windows can use it — it doesn't show up ready to go.
6. **Custom Script Extension needs the script in Blob Storage** — it can't be pointed at a file on the local machine.
7. **Installing the AD DS role and promoting to a domain controller are two separate steps** — the role alone does nothing until promotion runs.
8. **A Defender for Identity workspace provisions lazily**, on first access, not the instant a license is assigned.
9. **The newer sensor activation method depends on Defender for Endpoint device inventory** — the classic installer has no such dependency.
10. **Secure-by-default storage blocks public blob access even for internal lab use** — SAS tokens are the correct way around that, not disabling the security.
11. **"Stopped (deallocated)" is what actually stops billing** — just shutting down the OS from inside isn't enough.

---

**Next:** Project 8 — Email Security (Exchange + Defender for Office 365). Full roadmap: [`ROADMAP.md`](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab/blob/main/ROADMAP.md)
