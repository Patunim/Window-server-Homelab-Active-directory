# Active Directory Setup and Management

## Objective
This document outlines the process of setting up and managing Organizational Units (OUs), groups, and users within Active Directory (AD) using Windows Server. It serves as a guide to implementing a structured AD environment for organizational needs, including user rights management, resource access, and departmental groupings.

---

## Installing Windows Server in VirtualBox

Before beginning AD configuration, I installed Windows Server on a virtual machine using Oracle VirtualBox.

**Steps:**
1. Created a new VM in VirtualBox with 4GB RAM, 2 cores, and at least 80GB disk.
2. Mounted the Windows Server ISO and completed the installation.
3. Installed the Active Directory Domain Services (AD DS) role.

![Installation Screenshot](images/server-creation.png)

Installing Active Directory Domain Services (AD DS)
After setting up the base Windows Server installation, I proceeded to install Active Directory Domain Services (AD DS) to promote the server to a domain controller.

Here’s what I did:

I launched Server Manager from the Start menu.
Clicked on Manage > Add Roles and Features to start the setup wizard.
I selected Role-based or feature-based installation and chose my local server from the list.
On the Server Roles page, I checked Active Directory Domain Services. A pop-up appeared asking to add required features—I confirmed and continued.
I completed the wizard and clicked Install. Once the installation finished, a yellow notification bar appeared at the top of Server Manager.
I clicked Promote this server to a domain controller.
Since this was a fresh setup, I selected Add a new forest and entered eastcharmer.local as the root domain name.
On the next screen, I set a secure password for Directory Services Restore Mode (DSRM).
I accepted the default settings for the rest of the wizard and hit Install.
The server automatically restarted after installation. Once it came back up, my server was now the domain controller for eastcharmer.local.

[OU Creation](images/AD-installation.png)
---

## 1. Setting Up Organizational Units (OUs)
I started by organizing the Active Directory environment into various Organizational Units (OUs) to maintain a structured hierarchy within the domain. OUs are essential for delegating administrative control and applying group policies effectively.

### Create Main Organizational Units (OUs)
Right-click on the domain and select **New > Organizational Unit**. I created the following top-level OUs:
- USA
- Europe
- Asia

![OU Creation](images/Creation-of-USA,-Europe,-Asia-OUs.png)

### Create Sub-OUs for Departmental Groupings
Inside each main OU, I created sub-OUs for **Users**, **Computers**, and **Servers**.

**Example structure:**
```
USA
├── Users
├── Computers
└── Servers
```

![Sub-OUs](images/Sub-OUs-under-USA.png)

### Organize Resources Within OUs
User accounts and computer objects were placed in their respective OUs.

![OU Population](images/Populated-OUs-with-user-and-computer-objects.png)

---

## 2. Creating Active Directory Groups
Groups help manage permissions for users and computers. I created both **Security** and **Distribution** groups.

### Security Groups
Used to manage access to shared resources and assign permissions.
- Example: `IT Security Group` for IT staff

![Security Group](images/Opening-Active-Directory-Administrative-Center.png)

### Distribution Groups
Used for email communication.
- Example: `All Employees`, `Finance Team`

### Group Scopes
I selected scopes based on accessibility:
- **Global**: For domain-level groups
- **Universal**: For multi-domain groups
- **Domain Local**: For local domain resource access

---

## 3. Creating and Managing Users

### Create Users
User accounts were manually created under the appropriate OU.

**Example:**
- Name: EastFmer
- Logon: eastfmer@eastcharmer.local

![User Creation](images/User-creation-wizard.png)

### Automate User Creation
In production, I’d automate using PowerShell or CSV import to streamline onboarding.

### Assign Users to Groups
I added users to their respective department groups.

---

## 4. Assigning Permissions
I assigned folder/file access and configured rights based on roles.

### Assign Resource Permissions
- Finance had Modify access to their department folders
- Others had Read-Only access

![Permissions](images/Screenshot-showing-the-deny-all-access-setting.png)

### User Rights
User rights were defined for each security group:
- IT group had elevated access
- Standard users had limited access

![Policy Settings](images/Standard-Users-Policy-Settings.png)
![Admin Policy](images/Admin-Policy-Settings-Filled-In.png)

---

## 5. Hands-On Activity
To reinforce my knowledge, I completed a practical activity:

### Activity Steps:
1. Created test users and groups
2. Assigned permissions and tested access
3. Verified login scenarios and access rights

**Examples:**
- A user with insufficient rights received an **Access Denied** error.

![Access Denied](images/Access-Denied-message-on-Control-Panel-after-GPO-application.png)

- A user received a password rejection due to fine-grained policy.

![Password Rejected](images/Password-Rejected-Due-to-FGPP.png)

- Strong password creation succeeded.

![Strong Password](images/Successful-Strong-Password-Change.png)

- Locked account message observed.

![Account Locked](images/Account-Locked-Message.png)

---

## Summary
This Active Directory setup provides:
- Structured OU hierarchy
- Group-based access management
- Defined user rights and policy enforcement

Further configurations include password policies, GPO testing, and resource access simulations.

---

[More screenshots available in the repository under `/images`] 📁

