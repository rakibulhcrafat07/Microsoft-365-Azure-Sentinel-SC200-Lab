# Project 6 — Networking (Azure VNet, AZ-104)

**Phase 5 of the Zero-to-SOC lab.** AWS VPC comparison was dropped — this project stays 100% Azure/Microsoft, in line with the lab's scope. All resources were built in `RG-Phase1-Lab`, East US. All 91 screenshots from the working session are included below in chronological order, grouped by task.

## Tasks Completed
1. VNet with 2 subnets
2. NSG with inbound + outbound rules, associated to a subnet
3. Public IP address
4. VNet Peering (bidirectional, verified from both sides)
5. Azure Bastion — deployed and used for a real, verified connection to a test VM, then fully cleaned up

---

## 0. Starting Point

![Phase 5 task checklist](images/001.png)
![Repo state before this project](images/002.png)

## 1. VNet + 2 Subnets

Created `vnet-lab-sc200` (`10.10.0.0/16`, East US, `RG-Phase1-Lab`) with two subnets: `subnet-app` (`10.10.1.0/24`) and `subnet-data` (`10.10.2.0/24`).

![Empty Virtual networks list](images/003.png)
![Same, second view](images/004.png)
![Create virtual network — Basics tab, empty](images/005.png)
![Security tab — all paid add-ons left disabled](images/006.png)
![Address space tab — default 10.0.0.0/16 before editing](images/007.png)
![Editing default subnet → renaming/resizing to subnet-app](images/008.png)
![subnet-app saved as 10.10.1.0/24](images/009.png)
![Adding second subnet, subnet-data](images/010.png)
![Review + create — validation passed](images/011.png)
![Deployment complete](images/012.png)
![Verified: address space 10.10.0.0/16, 2 subnets](images/013.png)

## 2. NSG — Inbound / Outbound Rules

Created `nsg-app-subnet`, added an inbound SSH (22) rule restricted to my public IP, an outbound HTTPS (443) rule, then associated the NSG to `subnet-app`.

![Network security groups — empty list](images/014.png)
![Create NSG — Basics tab filled](images/015.png)
![Review + create — validation passed](images/016.png)
![NSG deployment complete](images/017.png)
![Checked public IP (103.55.146.57) before writing the inbound rule](images/018.png)
![Inbound rule form filled correctly (source IP, port 22, TCP, Allow)](images/019.png)
![Rule list — Add-SSH rule missing after first attempt (form closed without saving)](images/020.png)
![Public IP had changed to 103.55.146.61 (dynamic ISP IP) — had to redo the rule](images/021.png)
![Second attempt — wrong port (8080) and Protocol=Any by mistake](images/022.png)
![IP corrected to .61/32, protocol still wrong](images/023.png)
![Protocol corrected to TCP](images/024.png)
![Inbound rule Allow-SSH-MyIP confirmed in the rule list](images/025.png)
![Outbound rule form — Allow-HTTPS-Outbound, port 443](images/026.png)
![Outbound rule confirmed in the list, alongside Azure's default outbound rules](images/027.png)
![NSG → Subnets → Associate panel](images/028.png)
!["Successfully saved network security group for subnet 'subnet-app'"](images/029.png)
![Subnet list confirming NSG association](images/030.png)

## 3. Public IP Address

Created a Standard SKU, static public IP (`pip-lab-vm`) — first attempt had the wrong resource group (auto-selected `NetworkWatcherRG`), corrected to `RG-Phase1-Lab`.

![Create Public IP — wrong resource group caught before submitting](images/031.png)
![Review + create — validation passed, RG-Phase1-Lab](images/032.png)
![Deployment complete](images/033.png)
![Public IP overview — 20.231.234.225, Standard, Regional](images/034.png)

## 4. VNet Peering

Created a second VNet (`vnet-lab-sc200-peer`, `10.20.0.0/16` — a non-overlapping range) and peered it with `vnet-lab-sc200`. Verified the peering from **both** virtual networks.

![Virtual networks list — only vnet-lab-sc200 exists so far](images/035.png)
![Create second VNet — Basics tab](images/036.png)
![Address space set to 10.20.0.0/16](images/037.png)
![Editing default subnet — private-subnet option unchecked for consistency](images/038.png)
![Subnet renamed to subnet-peer-demo](images/039.png)
![Address space tab final state, 10.20.1.0/24 subnet](images/040.png)
![Second VNet deployment succeeded](images/041.png)
![Add peering form — remote VNet + settings filled](images/042.png)
![Add peering form — local peering link name filled](images/043.png)
![Add peering form complete, Add button active](images/044.png)
![Peering shows Connected + Fully Synchronized (viewed from vnet-lab-sc200)](images/045.png)
![Same peering, second confirmation view](images/046.png)
![Peering also shows Connected from vnet-lab-sc200-peer (bidirectional proof)](images/047.png)

## 5. Azure Bastion / JIT Access

Explored the Bastion quick-create configuration first, then deployed it for real, created a low-cost test VM with no public IP, and connected to it through the browser via Bastion.

![Bastion quick-create configuration reviewed (not yet deployed)](images/048.png)
![Bastion deployment started — "Creating a new Bastion"](images/049.png)

### Creating the test VM

Several retries were needed to find an available, low-cost VM size in this subscription/region.

![Virtual machines — empty list](images/050.png)
![Create VM — type selection dropdown](images/051.png)
![Basics tab — initial fields](images/052.png)
![Standard_B1s size shown as unavailable for this subscription](images/053.png)
![Size list — expensive D/E-series options shown by default](images/054.png)
![Searched "B2" sizes — most require a quota request](images/055.png)
![B2ats_v2 (free services eligible) selected](images/056.png)
![Accidentally clicked "Request quota" → landed on a support request page by mistake](images/057.png)
![Back on Select VM size — tooltip confirms B2ats_v2 is NotAvailableForSubscription](images/058.png)
![Browsing other size options](images/059.png)
![Series list cleared, browsing from scratch](images/060.png)
![B2ms searched — also unavailable for this subscription](images/061.png)
![D2s_v7 searched — available, Select button active](images/062.png)
![After clicking Select, session unexpectedly returned to the Azure Home page](images/063.png)
![Retried size selection — D2s_v7 again](images/064.png)
![Session dropped to Home page a second time](images/065.png)
![Third attempt — full Basics tab completed, size D2s_v7 retained this time](images/066.png)
![Disks tab — left at defaults](images/067.png)
![Networking tab — VNet/subnet auto-detected, existing NSG recognized](images/068.png)
![Networking tab — Public IP explicitly set to None (VM will rely on Bastion only)](images/069.png)
![Review + create — page 1 (price: $0.132 USD/hr, terms)](images/070.png)
![Review + create — page 2 (disks, networking, management settings)](images/071.png)
!["Generate new key pair" — SSH key pair prompt before deployment](images/072.png)
![VM deployment in progress](images/073.png)
![VM deployment complete](images/074.png)

### Connecting via Bastion

![Bastion status check — VM dropdown briefly empty right after VM creation](images/075.png)
![Connect via Bastion — form from the VM's own page](images/076.png)
![Authentication type dropdown — selecting "SSH Private Key from Local File"](images/077.png)
![**Connected — live shell: azureuser@vm-test-linux:~$**](images/078.png)

## Cleanup

Bastion and the test VM are billed hourly, so both were deleted immediately after verification, along with Bastion's auto-generated public IP. The resource group's pre-existing `Delete` lock (from Project 4 — RBAC) blocked the first delete attempt and had to be temporarily removed, then restored afterward.

![First VM delete attempt — confirmation dialog](images/079.png)
![VM delete failed: ScopeLocked (resource group has a Delete lock)](images/080.png)
![Resource group Locks — deleting the lock to proceed](images/081.png)
![Lock deleted successfully](images/082.png)
![VM delete executed successfully this time](images/083.png)
![Bastion overview page, about to delete](images/084.png)
![Activity log showing the Bastion delete operation in progress](images/085.png)
![Bastion resource returns 404 — confirms it's deleted](images/086.png)
![Public IP addresses list — locating Bastion's auto-created IP](images/087.png)
![Bastion's public IP deleted successfully](images/088.png)
![Delete lock re-created on RG-Phase1-Lab, restoring the Project 4 protection](images/089.png)

## Wrap-up

![Final check of the checklist / repo state](images/090.png)
![GitHub repo as it stood going into this project](images/091.png)

---

## Mistakes & Fixes

| Issue | Fix |
|---|---|
| Home ISP assigned a **dynamic public IP** — it changed mid-session (`103.55.146.57` → `.61`), breaking the NSG inbound rule | Updated the rule with the new IP; noted this as the real-world reason Bastion/VPN beats IP-based NSG rules |
| First inbound NSG rule attempt silently failed to save (form closed without adding the rule) | Recreated the rule, confirmed with Refresh before moving on |
| Wrong port (8080) and Protocol=Any entered by mistake while redoing the inbound rule | Corrected to port 22 / TCP before submitting |
| Public IP creation defaulted to `NetworkWatcherRG` instead of the lab's resource group | Manually reselected `RG-Phase1-Lab` |
| `Standard_B1s`, `Standard_B2ats_v2`, and `Standard_B2ms` all returned `NotAvailableForSubscription` in East US | Searched sizes without a "Request quota" link; settled on `Standard_D2s_v7` |
| Accidentally clicked "Request quota" instead of the VM size name, landing on an unrelated support-ticket page | Navigated back and reselected the size row correctly |
| VM creation wizard twice dropped back to the Azure Home page after selecting a size (session/navigation glitch) | Restarted the VM creation flow from scratch each time; size choice was remembered on the third attempt |
| VM `Delete` failed with `ScopeLocked` | Temporarily removed the `PreventDelete-RG-Phase1` lock, deleted the VM, then restored the same lock |
| Bastion's own public IP is **not** auto-deleted when Bastion is deleted | Deleted it manually as a separate step |

## Key Learnings

- A subnet-level NSG rule (like `Allow-SSH-MyIP`) applies automatically to any VM later placed in that subnet — Azure surfaced this during VM creation ("subnet is already associated to nsg-app-subnet"), so no VM-level NSG was needed.
- Azure's default NSG rules differ by direction: inbound ends in an implicit **Deny All**, while outbound already has an implicit **Allow Internet** — outbound rules need less manual work than inbound by default.
- VNet Peering must be verified from **both** virtual networks — each side has its own peering link name and status, and both need to show "Connected."
- Azure Bastion removes the need for a VM to have any public IP at all: connectivity works entirely over the VM's private IP, tunneled through the browser over HTTPS.
- Bastion pricing is hourly and starts the moment deployment succeeds — cleanup must happen immediately after verification, and its auto-created public IP is a separate billable resource that survives Bastion's own deletion.
- A resource-group-level `Delete` lock (set up in an earlier RBAC project) blocks *all* resource deletions in that group, not just the one it was meant to protect — it has to be temporarily lifted for any cleanup work, then restored.

---

*Part of [Zero-to-SOC: A Self-Funded, Multi-Certification Cloud Security Lab](https://github.com/rakibulhcrafat07/Microsoft-365-Azure-Sentinel-SC200-Lab)*
