**Installing Windows Server in VirtualBox and Configuring Active Directory**

Before configuring Active Directory, I set up a virtualized lab using Oracle VirtualBox.

---

### VM Setup in VirtualBox
- Created a new Virtual Machine.
- Selected the Windows Server 2022 ISO as the installation medium.
- Assigned 2 GB RAM and 1 CPU core.
- Installed Windows Server OS with Desktop Experience.
- Set a secure administrator password.
- Set the network adapter to "Internal Network."
- Assigned a static IP address.

![VM Settings](images/VirtualBox-VM-Settings-Windows-Server-Desktop.png)

---

### Active Directory: OUs, Users, Groups & Permissions

#### Organizational Units (OUs)
I structured Active Directory with top-level OUs per region (USA, Europe, Asia) and created sub-OUs inside each for Users, Computers, and Servers.

- **Example:**
  - USA > Users
  - USA > Computers
  - USA > Servers

📸 [Insert Screenshot: Creation of USA, Europe, Asia OUs] — `images/OU-Creation-USA-Europe-Asia.png`
📸 [Insert Screenshot: Sub-OUs under USA] — `images/Sub-OUs-USA.png`
📸 [Insert Screenshot: Populated OUs with user and computer objects] — `images/Objects-In-OUs.png`

#### Groups and Group Scopes
Created security groups (e.g., IT-SecurityGroup, HR-SGroup) for permission assignment.

- Group Scopes:
  - Global: Same domain
  - Universal: Cross-domain (forests)
  - Domain Local: Within domain only

📸 [Insert Screenshot: Security group creation window] — images/Security-Group-Creation.png
📸 [Insert Screenshot: Group scope options dropdown] — `images/Group-Scope-Options.png`

#### Users
Manually created users `Joshua` and `vboxuser`, and assigned them to OUs and groups under the domain `Patrick.com`.
- Example: `joshua@patrick.com`, `vboxuser@patrick.com`
- Password policy: Complexity, expiration, lockout settings enforced.

📸 [Insert Screenshot: User creation wizard] — `images/User-Creation-Wizard.png`
📸 [Insert Screenshot: Password policy settings or Fine-Grained Policy summary] — `images/Password-Policy-Settings.png`
📸 [Insert Screenshot: User group membership tab] — `images/User-Group-Membership.png`

---

### Group Policy Objects (GPOs)

#### GPO 1: Password Policy + Account Lockout Policy
Edited the Default Domain Policy to apply a secure password policy to all domain users in `Patrick.com`:
- Minimum password length: 12
- Enforced complexity and history
- Max age: 90 days

Also configured lockout policy:
- Threshold: 3 failed logins
- Lockout duration: 30 minutes
- Reset counter: 30 minutes

Location:
`Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies`

📸 [Screenshot of configured password policy settings] — `images/GPO-Password-Policy.png`
📸 [Insert Screenshot – Password Rejected Due to Weakness] — `images/Test-Weak-Password-Rejected.png`
📸 [Insert Screenshot – Successful Strong Password Change] — `images/Test-Strong-Password-Accepted.png`
📸 [Insert Screenshot – Account Lockout Policy Settings] — `images/Account-Lockout-Policy.png`
📸 [Insert Screenshot – Account Locked Message] — `images/Account-Locked-Message.png`

#### GPO 2: Drive Mapping
Automatically maps `S:` to `\\ServerName\SharedFolder`

Location:
`User Configuration → Preferences → Windows Settings → Drive Maps`

📸 [Screenshot of drive map creation dialog] — `images/GPO-Drive-Map.png`

#### GPO 3: Desktop Wallpaper
Applies a uniform wallpaper via `\\ServerName\Wallpapers\company_wallpaper.jpg`

📸 [Screenshot showing the wallpaper path configuration] — `images/GPO-Wallpaper-Settings.png`

#### GPO 4: Restrict Control Panel Access
Prevents users from accessing system settings.

📸 [Screenshot showing the restriction setting enabled] — `images/GPO-Restrict-Control-Panel.png`

#### GPO 5: Disable USB Storage Devices
Blocks access to USB drives.

📸 [Screenshot showing the deny all access setting] — `images/GPO-Disable-USB.png`

#### GPO Assignments:
| GPO Name               | Type                  | Linked OU       |
|------------------------|-----------------------|-----------------|
| Restrict Control Panel | User Configuration    | USA > Users     |
| Password Policy        | Computer Configuration| USA > Computers |
| Drive Mapping          | User Configuration    | USA > Users     |
| Disable USB Devices    | Computer Configuration| USA > Computers |
| Set Wallpaper          | User Configuration    | USA > Users     |

#### GPO Testing
After joining the Windows 10 client machine named `client` to the domain and moving its computer object to `USA > Computers`, I tested GPOs via `gpupdate /force`.

📸 [Insert Screenshot: Active Directory Users and Computers showing the computer account under default Computers container] — `images/ADUC-Computer-Moved.png`
📸 [Insert Screenshot: Command Prompt window showing gpupdate force output] — `images/GPUpdate-Command.png`
📸 [Insert Screenshot: Access Denied message on Control Panel after GPO application] — `images/Control-Panel-Access-Denied.png`

---

### File Server Resource Manager (FSRM)

#### Installation
Installed FSRM role via Server Manager > Add Roles and Features.

📸 [Insert Screenshot: Add Roles and Features wizard with File Server Resource Manager selected] — `images/FSRM-Installation.png`

#### Quota Management
- Applied 10 GB quota to `C:\Shares\DeptShared`
- Notification at 80% usage

📸 [Insert Screenshot: Quota path selection for shared folder] — `images/FSRM-Quota-Path.png`
📸 [Insert Screenshot: Custom quota definition page] — `images/FSRM-Custom-Quota.png`
📸 [Insert Screenshot: Notification threshold settings screen] — `images/FSRM-Threshold-Notification.png`

#### File Screening
- Blocked Audio, Video, Executables, Compressed Files, etc.
- Allowed only Document and Text Files
- Named policy: `Shared_Dept_FileScreen_Policy`

📸 [Insert Screenshot: File screen folder path selection] — `images/FSRM-File-Screen-Path.png`
📸 [Insert Screenshot: File types selected for blocking] — `images/FSRM-File-Groups.png`
📸 [Insert Screenshot: Custom file screen template with selected settings] — `images/FSRM-Policy-Summary.png`

---

### User Rights Assignment

#### What I Did
- Denied log on locally for HR group
- Allowed RDP for IT group

Location:
`Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment`

📸 [Insert Screenshot – Deny Log On Locally Policy] — `images/User-Rights-Deny-Logon.png`
📸 [Insert Screenshot – Remote Desktop Logon Policy] — `images/User-Rights-Allow-RDP.png`

---

### Fine-Grained Password Policies (FGPP)

#### Admin Password Policy
- Name: `Admin_PasswordPolicy`
- Min length: 15
- Lockout threshold: 3
- Group: IT Admins

📸 [Insert Screenshot – Admin Policy Settings Filled In] — `images/Admin-Policy-Settings-Filled-In.png`
📸 [Insert Screenshot – Adding IT Admins Group to the Policy] — `images/Add-IT-Group-To-Policy.png`

#### Standard Users Policy
- Name: `StandardUsers_PasswordPolicy`
- Min length: 10
- Group: Domain Users

📸 [Insert Screenshot – Standard Users Policy Settings] — `images/Standard-Users-Policy-Settings.png`

#### Precedence Notes
- Lower number = higher priority
- Users in both policies get `Admin_PasswordPolicy`

📸 [Insert Screenshot – Password Rejected Due to FGPP] — `images/FGPP-Policy-Precedence.png`

