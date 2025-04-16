Installing Windows Server in VirtualBox and Configuring Active Directory

Before configuring Active Directory, I set up a virtualized lab using Oracle VirtualBox.

VM Setup in VirtualBox

Created a new Virtual Machine.

Selected the Windows Server 2022 ISO as the installation medium.

Assigned 2 GB RAM and 1 CPU core.

Installed Windows Server OS with Desktop Experience.

Set a secure administrator password.

Set the network adapter to "Internal Network."

Assigned a static IP address.

![VM Settings](images/virtualbox vm settings windows server desktop.png)

Active Directory: OUs, Users, Groups & Permissions

Organizational Units (OUs)

I structured Active Directory with top-level OUs per region (USA, Europe, Asia) and created sub-OUs inside each for Users, Computers, and Servers.

Example:

USA > Users

USA > Computers

USA > Servers

📸 [Insert Screenshot: Creation of USA, Europe, Asia OUs] — images/ou creation usa europe asia.png
📸 [Insert Screenshot: Sub-OUs under USA] — images/sub ou usa.png
📸 [Insert Screenshot: Populated OUs with user and computer objects] — images/objects in ous.png

Groups and Group Scopes

Created security groups (e.g., IT-SecurityGroup, HR-SGroup) for permission assignment.

Group Scopes:

Global: Same domain

Universal: Cross-domain (forests)

Domain Local: Within domain only

📸 [Insert Screenshot: Security group creation window] — images/security group creation.png
📸 [Insert Screenshot: Group scope options dropdown] — images/group scope options.png

Users

Manually created users Joshua and vboxuser, and assigned them to OUs and groups under the domain Patrick.com.

Example: joshua@patrick.com, vboxuser@patrick.com

Password policy: Complexity, expiration, lockout settings enforced.

📸 [Insert Screenshot: User creation wizard] — images/user creation wizard.png
📸 [Insert Screenshot: Password policy settings or Fine-Grained Policy summary] — images/password policy settings.png
📸 [Insert Screenshot: User group membership tab] — images/user group membership.png

Group Policy Objects (GPOs)

GPO 1: Password Policy + Account Lockout Policy

Edited the Default Domain Policy to apply a secure password policy to all domain users in Patrick.com:

Minimum password length: 12

Enforced complexity and history

Max age: 90 days

Also configured lockout policy:

Threshold: 3 failed logins

Lockout duration: 30 minutes

Reset counter: 30 minutes

Location:
Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies

📸 [Screenshot of configured password policy settings] — images/gpo password policy.png
📸 [Insert Screenshot – Password Rejected Due to Weakness] — images/test weak password rejected.png
📸 [Insert Screenshot – Successful Strong Password Change] — images/test strong password accepted.png
📸 [Insert Screenshot – Account Lockout Policy Settings] — images/account lockout policy.png
📸 [Insert Screenshot – Account Locked Message] — images/account locked message.png

GPO 2: Drive Mapping

Automatically maps S: to \\ServerName\SharedFolder

Location:
User Configuration → Preferences → Windows Settings → Drive Maps

📸 [Screenshot of drive map creation dialog] — images/gpo drive map.png

GPO 3: Desktop Wallpaper

Applies a uniform wallpaper via \\ServerName\Wallpapers\company_wallpaper.jpg

📸 [Screenshot showing the wallpaper path configuration] — images/gpo wallpaper settings.png

GPO 4: Restrict Control Panel Access

Prevents users from accessing system settings.

📸 [Screenshot showing the restriction setting enabled] — images/gpo restrict control panel.png

GPO 5: Disable USB Storage Devices

Blocks access to USB drives.

📸 [Screenshot showing the deny all access setting] — images/gpo disable usb.png

### GPO Assignments

| GPO Name               | Configuration Type     | Linked OU        |
|------------------------|------------------------|------------------|
| Restrict Control Panel | User Configuration     | USA > Users      |
| Password Policy        | Computer Configuration | USA > Computers  |
| Drive Mapping          | User Configuration     | USA > Users      |
| Disable USB Devices    | Computer Configuration | USA > Computers  |
| Set Wallpaper          | User Configuration     | USA > Users      |

GPO Testing

After joining the Windows 10 client machine named client to the domain and moving its computer object to USA > Computers, I tested GPOs via gpupdate /force.

📸 [Insert Screenshot: Active Directory Users and Computers showing the computer account under default Computers container] — images/aduc computer moved.png
📸 [Insert Screenshot: Command Prompt window showing gpupdate force output] — images/gpupdate command.png
📸 [Insert Screenshot: Access Denied message on Control Panel after GPO application] — images/control panel access denied.png

File Server Resource Manager (FSRM)

Installation

Installed FSRM role via Server Manager > Add Roles and Features.

📸 [Insert Screenshot: Add Roles and Features wizard with File Server Resource Manager selected] — images/fsrm installation.png

Quota Management

Applied 10 GB quota to C:\Shares\DeptShared

Notification at 80% usage

📸 [Insert Screenshot: Quota path selection for shared folder] — images/fsrm quota path.png
📸 [Insert Screenshot: Custom quota definition page] — images/fsrm custom quota.png
📸 [Insert Screenshot: Notification threshold settings screen] — images/fsrm threshold notification.png

File Screening

Blocked Audio, Video, Executables, Compressed Files, etc.

Allowed only Document and Text Files

Named policy: Shared_Dept_FileScreen_Policy

📸 [Insert Screenshot: File screen folder path selection] — images/fsrm file screen path.png
📸 [Insert Screenshot: File types selected for blocking] — images/fsrm file groups.png
📸 [Insert Screenshot: Custom file screen template with selected settings] — images/fsrm policy summary.png

User Rights Assignment

What I Did

Denied log on locally for HR group

Allowed RDP for IT group

Location:
Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment

📸 [Insert Screenshot – Deny Log On Locally Policy] — images/user rights deny logon.png
📸 [Insert Screenshot – Remote Desktop Logon Policy] — images/user rights allow rdp.png

Fine-Grained Password Policies (FGPP)

Admin Password Policy

Name: Admin_PasswordPolicy

Min length: 15

Lockout threshold: 3

Group: IT Admins

📸 [Insert Screenshot – Admin Policy Settings Filled In] — images/fgpp admin settings.png
📸 [Insert Screenshot – Adding IT Admins Group to the Policy] — images/fgpp add it group.png

Standard Users Policy

Name: StandardUsers_PasswordPolicy

Min length: 10

Group: Domain Users

📸 [Insert Screenshot – Standard Users Policy Settings] — images/fgpp standard user.png

Precedence Notes

Lower number = higher priority

Users in both policies get Admin_PasswordPolicy

📸 [Insert Screenshot – Password Rejected Due to FGPP] — images/fgpp policy precedence.png

