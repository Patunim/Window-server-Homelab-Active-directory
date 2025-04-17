### VM Setup in VirtualBox
- Created a new Virtual Machine.
- Selected the Windows Server 2022 ISO as the installation medium.
- Assigned 2 GB RAM and 1 CPU core.
- Installed Windows Server OS with Desktop Experience.
- Set a secure administrator password.
- Set the network adapter to "Internal Network."
- Assigned a static IP address.

![VM Settings](images/server-creation.png)

### Windows 10 Client Setup and Domain Join

#### VM Creation and Configuration
- Created a new virtual machine for the Windows 10 client in VirtualBox
- Installed Windows 10 Pro using ISO
- Assigned 2 GB RAM, 1 CPU core
- Set network adapter to **Internal Network**

![Windows 10 VM settings](images/Win10-VM-Settings.png)
---
Active Directory Installation

I installed the Active Directory Domain Services (AD DS) role using the Server Manager:

Opened Server Manager > Add Roles and Features.

Selected Role-based or feature-based installation.

Chose the local server as the destination.

Selected Active Directory Domain Services from the server roles.

Completed the wizard and restarted the server when prompted.

After the role installation:
Opened Server Manager and clicked the notification flag.
Selected Promote this server to a domain controller.
Created a new forest named patrick.com
![VM Settings](images/AD-installation.png)
Set a Directory Services Restore Mode (DSRM) password.
Completed the configuration and rebooted the server.

After reboot:
Verified the server is now a domain controller.
Logged in as patrick\Administrator.
Confirmed the presence of Active Directory tools (ADUC, DNS, etc).


### Active Directory: OUs, Users, Groups & Permissions

#### Organizational Units (OUs)
I structured Active Directory with top-level OUs per region (USA, Europe, Asia) and created sub-OUs inside each for Users, Computers, and Servers.

- **Example:**
  - USA > Users
  - USA > Computers
  - USA > Servers

![Creation of USA, Europe, Asia OUs](images/Creation-of-USA,-Europe,-Asia-OUs.png)
![Sub-OUs under USA](images/Sub-OUs-under-USA.png)

#### Users
Manually created users `Joshua` and `vboxuser`, and assigned them to OUs and groups under the domain `Patrick.com`.
- Example: `joshua@patrick.com`, `vboxuser@patrick.com`
- Password policy: Complexity, expiration, lockout settings enforced.

![User creation wizard](images/User-creation-wizard.png)
![Populated OUs with user and computer objects](images/Populated-OUs-with-user-and-computer-objects.png)
![Computer account moved](images/Active-Directory-Users-and-Computers-showing-the-computer-account-under-default-Computers--container.png)

#### Groups and Group Scopes
Created security groups (e.g., IT-SecurityGroup, HR-SGroup) for permission assignment.

- Group Scopes:
  - Global: Same domain
  - Universal: Cross-domain (forests)
  - Domain Local: Within domain only


#### Network Configuration
- Assigned static IP in the same subnet as the domain controller
- Set Preferred DNS to DC IP

**Example:**
- IP: `192.168.10.20`
- Subnet: `255.255.255.0`
- Default Gateway: `192.168.10.1`
- Preferred DNS: `192.168.10.10`

![Windows 10 network settings](images/Win10-Network-Settings.png)

#### Joining the Domain
- Verified connectivity to the domain controller
- Joined the domain `patrick.com` using domain credentials

![Domain join dialog](images/Win10-Domain-Join.png)
![Domain join success](images/Win10-Domain-Join-Success.png)
---

### Group Policy Objects (GPOs)
#### GPO Assignments:
| GPO Name               | Type                  | Linked OU       |
|------------------------|-----------------------|-----------------|
| Restrict Control Panel | User Configuration    | USA > Users     |
| Password Policy        | Computer Configuration| USA > Computers |
| Drive Mapping          | User Configuration    | USA > Users     |
| Disable USB Devices    | Computer Configuration| USA > Computers |
| Set Wallpaper          | User Configuration    | USA > Users     |

#### GPO 1: Password Policy + Account Lockout Policy
Edited the Default Domain Policy to apply a secure password policy to all domain users in `Patrick.com`:
- Minimum password length: 12
- Enforced complexity and history
- Max age: 90 days
![Password policy settings or Fine-Grained Policy summary](images/Screenshot-of-configured-password-policy-settings.png)


Also configured lockout policy:
Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies
- Threshold: 3 failed logins
- Lockout duration: 30 minutes
- Reset counter: 30 minutes
![Account Lockout Policy Settings](images/Account-Lockout-Policy-Settings.png)

![Account Locked Message](images/Account-Locked-Message.png)

#### GPO 2: Drive Mapping
Automatically maps S:to \\ServerName\SharedFolder

Location:
User Configuration → Preferences → Windows Settings → Drive Maps

![Drive map creation dialog](images/Screenshot-of-drive-map-creation-dialog.png)

#### GPO 3: Desktop Wallpaper
Applies a uniform wallpaper via `\\ServerName\Wallpapers\company_wallpaper.jpg`

![Wallpaper path configuration](images/Screenshot-showing-the-wallpaper-path-configuration.png)

#### GPO 4: Restrict Control Panel Access
Prevents users from accessing system settings.

![Restriction setting enabled](images/Screenshot-showing-the-restriction-setting-enabled.png)

#### GPO 5: Disable USB Storage Devices
Blocks access to USB drives.

![Deny all access setting](images/Screenshot-showing-the-deny-all-access-setting.png)




🔹 5. GPO Testing and Validation
To test the "Restrict Control Panel" policy:
Logged into the domain-joined client using a test domain user.
Attempted to open the Control Panel — it was accessible initially.
Launched Command Prompt as Administrator and ran:

`gpupdate /force`
Waited for the policy update confirmation.

Re-attempted to access Control Panel — access was denied as expected.
Screenshot Placeholder:
![gpupdate force output](images/Command-Prompt-window-showing-gpupdate-force-output.png)
![Access Denied on Control Panel](images/Access-Denied-message-on-Control-Panel-after-GPO-application.png)
✅ Result: GPO successfully applied and functional.


---

### File Server Resource Manager (FSRM)

#### Installation
Installed FSRM role via Server Manager > Add Roles and Features.

![FSRM installation](images/Add-Roles-and-Features-wizard-with-File-Server-Resource-Manager-selected.png)
![Custom file screen template](images/FSRM-appearing-under-Administrative-Tools.png)

#### Quota Management
- Applied 10 GB quota to `C:\Shares\DeptShared`
- Notification at 80% usage

![Quota path selection](images/Quota-path-selection-for-shared-folder.png)
![Custom quota definition](images/Custom-quota-definition-page.png)
![Notification threshold settings](images/Notification-threshold-settings-screen.png)

#### File Screening
- Blocked Audio, Video, Executables, Compressed Files, etc.
- Allowed only Document and Text Files
- Named policy: `Shared_Dept_FileScreen_Policy`

![File screen path selection](images/FSRM-appearing-under-Administrative-Tools.png)
![File types selected for blocking](images/File-types-selected-for-blocking.png)


---

Password Policy Configuration (Group Policy Management)
In this part of the lab, I configured and enforced a strong password policy for all Active Directory users. This is foundational for securing accounts and protecting against unauthorized access.

🧠 What I Did
I edited the Default Domain Policy to apply a secure password policy to all domain users. This includes setting:

Minimum password length

Password complexity

Password history

Password expiration (age)

The default minimum password length in Windows Server is 8 characters, which is outdated. I increased it to 12 characters for stronger protection in today's digital environment.

🔧 Steps I Followed
Open Group Policy Management Console (GPMC)

Start → Administrative Tools → Group Policy Management

Edit the Default Domain Policy

Navigated to:
Forest > Domains > yourdomain.local > Default Domain Policy

Right-clicked and selected Edit

📸 [Insert Screenshot – Editing Default Domain Policy]

Navigate to Password Policy

Inside Group Policy Management Editor:

Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy

📸 [Insert Screenshot – Password Policy Location]

Configured the following settings:

Enforce password history: 5 passwords remembered

Maximum password age: 90 days

Minimum password length: 12 characters

Password must meet complexity requirements: Enabled
(Includes uppercase, lowercase, numbers, symbols)

![Password policy settings or Fine-Grained Policy summary](images/Screenshot-of-configured-password-policy-settings.png)

✅ Testing the Policy
To test if the new policy was enforced:

Created a test user in Active Directory

Checked “User must change password at next logon”

Logged in as the test user and tried setting a weak password: 1234567

![Password Rejected Due to Weakness](images/Password-Rejected-Due-to-FGPP.png)

Then, I set a strong password meeting all complexity rules. It was accepted, confirming that the policy was active.

![Successful Strong Password Change](images/Successful-Strong-Password-Change.png)

🔒 Account Lockout Policy
Next, I configured the Account Lockout Policy to protect against brute force attacks by locking accounts after a few failed login attempts.

🛠️ Configuration Steps
Edited the Default Domain Policy again (same steps as above)

Navigated to: Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy

Set the following:

Account lockout threshold: 3 invalid attempts

Account lockout duration: 30 minutes

Reset account lockout counter after: 30 minutes

📸 [Insert Screenshot – Account Lockout Policy Settings]

✅ Testing Account Lockout
Logged in as a test user

Entered wrong passwords 3 times

Got an error that the account was locked out

📸 [Insert Screenshot – Account Locked Message]




### User Rights Assignment

#### What I Did
- Denied log on locally for HR group
- Allowed RDP for IT group

Location:
Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment

![Deny Log On Locally Policy](images/Deny-Log-On-Locally-Policy.png)
![Remote Desktop Logon Policy](images/Remote-Desktop-Logon-Policy.png)

---

Fine-Grained Password Policies (FGPP)
I used Active Directory Administrative Center (ADAC) to configure fine-grained password policies.

🧭 Steps:
Opened Active Directory Administrative Center from the Start menu.

Navigated to System > Password Settings Container.

Right-clicked and selected New → Password Settings to create a custom policy.

Assigned the policy to a specific security group by selecting Add → Browse → Groups.

Adjusted precedence to ensure the correct policy applies when users belong to multiple groups.

![Admin Policy Settings](images/Opening-Active-Directory-Administrative-Center.png)
![Admin Policy Settings](images/Navigating-to-Password-Settings-Container.png)
📸 Screenshot Placeholder: Creating new Password Settings policy




### Fine-Grained Password Policies (FGPP)

#### Admin Password Policy
- Name: `Admin_PasswordPolicy`
- Min length: 15
- Lockout threshold: 3
- Group: IT Admins

![Admin Policy Settings](images/Admin-Policy-Settings-Filled-In.png)


#### Standard Users Policy
- Name: `StandardUsers_PasswordPolicy`
- Min length: 10
- Group: Domain Users

![Standard Users Policy Settings](images/Standard-Users-Policy-Settings.png)

#### Precedence Notes
- Lower number = higher priority
- Users in both policies get `Admin_PasswordPolicy`

![Password Rejected Due to FGPP](images/Password-Rejected-Due-to-FGPP.png)

---



#### Post-Domain Join Steps
- Logged in as `joshua@patrick.com`
- Verified group policies applied (wallpaper, drive, restrictions)
- Ran `gpupdate /force` and rebooted

![Domain login screen](images/Win10-Domain-Login.png)
![gpupdate command run](images/Win10-gpupdate-force.png)
![Wallpaper and drive mapping applied](images/Win10-Wallpaper-DriveMap.png)
