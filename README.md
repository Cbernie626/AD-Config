# 🎬 Active Directory Lab Series: Domain Controller & Client Setup (DHCP, DNS, AD)

## ✨ Introduction: Building Your Active Directory Foundation

Welcome to the foundational repository for the Active Directory Lab Series!

This guide provides the complete, structured blueprint for setting up a robust, working Active Directory environment from the ground up using virtualization. **Active Directory (AD)** is the core directory service for Windows domain networks, and learning it hands-on is essential for any aspiring or current IT professional.

This lab focuses on the initial, critical build-out: configuring a single Windows Server to function as your **Domain Controller (DC)**, handling core services like **DHCP** (Dynamic Host Configuration Protocol), **DNS** (Domain Name System), and **NAT** (Network Address Translation). By the end, you will have a client machine successfully joined to your custom domain, ready for deep dives into Group Policy, user management, and security that will follow in later parts of the series.

Whether you're studying for certifications, practicing real-world administration skills, or just exploring server technologies, this repository is your roadmap.

## 📺 Video Guide

A full, step-by-step video guide for this setup is available on my YouTube channel:
[Active Directory Lab Series: Domain Controller & Client Setup (DHCP, DNS, AD)](https://www.youtube.com/watch?v=qfLQRlMKzNU)

---

## ⚙️ Lab Topology Overview

The lab consists of a single **Domain Controller (DC)** and a **Client Machine**, connected on an isolated internal network. The DC acts as a router, DHCP server, and DNS server, allowing the client to access the internet through Network Address Translation (NAT).

| Component | Role | Network Connection | Configuration Details |
| :--- | :--- | :--- | :--- |
| **Domain Controller (DC)** | AD DS, DHCP, DNS, NAT (via RAS) | **NIC 1 (External/Internet):** Bridged Adapter | IP assigned by Host Router DHCP |
| | | **NIC 2 (Internal/Lab):** Internal Network | **Static IP:** `172.16.0.1` |
| **Client Machine** | Windows 11 Client | Internal Network | **DHCP Assigned:** `172.16.0.x` |
| **Domain Name** | My virtual network identity | N/A | `mydomain.com` |

---

## 💻 Prerequisites

To follow this guide, you will need:

* **VirtualBox** (or a similar hypervisor) and the **Extension Pack**.
* **Windows Server 2025** ISO
* **Windows 11** ISO

### Recommended VM Specifications

| VM Name | OS | CPU Cores | RAM | Storage |
| :--- | :--- | :--- | :--- | :--- |
| **DC** | Windows Server 2025 | 4 | 3 GB | 100 GB |
| **Client-01** | Windows 11 | 2 | 2 GB | 50 GB |

---

Here is the full GitHub Markdown for your detailed "Key Setup Steps" section. You can paste this directly into your README.md file.

Markdown

# 🛠️ Key Setup Steps (Detailed)

This guide provides the complete, step-by-step configuration to build your Domain Controller and integrate the client machine.

---

## 1. 🖥️ Domain Controller Initial Setup

The goal is to prepare the **DC** (Domain Controller) with correct networking before promoting it to a domain controller role.

* **Install Base OS:** Install **Windows Server 2025** (choosing the **Desktop Experience** version).
* **Install VM Tools:** Install **Guest Editions** (or equivalent VM tools) to enable features like seamless mouse integration, shared clipboard, and improved screen resolution/performance.
* **Rename the PC:** Rename the computer to **`DC`**. A restart will be required after this change.
* **Configure Network Adapters:** The server uses two Network Interface Cards (NICs):
    * **External NIC:** Connect this to a **Bridged Adapter** network in VirtualBox. Rename this adapter to **`Internet`**. It'll automatically receive an IP address from your home router.
    * **Internal NIC:** Connect this to an **Internal Network** in VirtualBox (e.g., named `ADLabNet`). Rename this adapter to **`Internal`**.
        * Manually configure the static IP settings for the **`Internal`** adapter:
            * **IP Address:** `172.16.0.1`
            * **Subnet Mask:** `255.255.255.0`
            * **Default Gateway:** (Leave blank)
            * **Preferred DNS:** `127.0.0.1` (Uses the server itself for DNS).

---

## 2. 🛡️ Active Directory Domain Services (AD DS)

This step transforms the server into the heart of your network—the Domain Controller.

* **Install AD DS Role:** Use **Server Manager** to **Add Roles and Features**. Select and install the **Active Directory Domain Services** role.
* **Promote the Server:** After the role installation is complete, click the notification flag in Server Manager and select **Promote this server to a domain controller**.
    * Choose **Add a new forest**.
    * Set the **Root domain name** to **`mydomain.com`**.
    * Follow the prompts and install. The DC will restart automatically.
* **Create Admin User:** Once logged back in, go to **Tools** > **Active Directory Users and Computers (ADUC)**.
    * Create a new **Organizational Unit (OU)** for administrators (e.g., `Admins`).
    * Create a new User account (e.g., **`a-cbernie`**) within the `Admins` OU. The **`a-`** prefix signifies this is a dedicated administrator account.
    * Right-click the new user, select **Properties**, go to the **Member Of** tab, and add the user to the **`Domain Admins`** group.

---

## 3. 🌐 Network Services (NAT, DHCP)

These services enable client machines to receive automatic IP configurations and access the internet.

* **Configure NAT/Routing:**
    * Install the **Remote Access (RAS)** role (select only **Routing** for the Role Services).
    * Open **Tools** > **Routing and Remote Access**. Right-click the server name and choose **Configure and Enable Routing and Remote Access**.
    * Select the **Network address translation (NAT)** option.
    * Select your **`Internet`** NIC as the public interface.
* **Configure DHCP Server:**
    * Install the **DHCP Server** role via Server Manager.
    * Go to **Tools** > **DHCP**. Expand your server and IPv4.
    * Right-click IPv4 and select **New Scope**.
    * **Create a new DHCP Scope:**
        * **Start IP:** `172.16.0.100`
        * **End IP:** `172.16.0.200` (The range of IPs clients can receive).
    * **Configure Scope Options (crucial):**
        * **Router (Default Gateway):** Set to the DC's internal static IP: **`172.16.0.1`**.
        * **DNS Servers:** Verify that the DC's static internal IP (`172.16.0.1`) is listed as the primary DNS server.
    * **Authorize the Scope:** Right-click the server name in the DHCP console and select **Authorize**.

---

## 4. 👥 Bulk User Creation Script

This step adds a large number of test users for GPO and user management practice.

* **Prepare Files:** Ensure the **`users.ps1`** script and the **`names.txt`** file are located in the same directory on the DC.
* **Run the Script:** Open **PowerShell ISE** (as Administrator). Load and run the **`users.ps1`** script.
* **Verify:** Check **ADUC** to confirm the new **`_users`** OU is present and populated with the new accounts.

> **Note:** The actual PowerShell script file should be included in your GitHub repository and linked here (e.g., `[users.ps1](users.ps1)`).

---

## 5. 💻 Client Integration

The final step is connecting the client to the newly configured domain.

* **Create and Configure Client VM:**
    * Create a new VM using the **Windows 11 ISO**.
    * Set its network adapter to the same **Internal Network** (e.g., `ADLabNet`) used by the DC's `Internal` NIC.
* **Verify Connectivity (DHCP & DNS):**
    * Open Command Prompt (`cmd`).
    * Run `ipconfig` to confirm the client received an IP address in the `172.16.0.100` to `172.16.0.200` range.
    * Test DNS and NAT functionality: **`ping google.com`** and **`ping mydomain.com`**.
* **Join the Domain:**
    * Go to **Settings** > **System** > **About** > **Domain or workgroup**.
    * Click **Change** and switch the computer to a **Domain**.
    * Enter the domain name: **`mydomain.com`**.
    * Enter your administrative credentials (**`a-cbernie`**) when prompted.
    * Restart the Windows 11 client.
* **Final Login & Verification:**
    * At the login screen, select **Other user**.
    * Log in using the domain account: **`a-cbernie`**.
    * On the DC, check **Tools** > **DHCP** under **Address Leases** to confirm the client's hostna
