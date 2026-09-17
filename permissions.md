# 🔐 Windows NTFS Permissions

> A hands-on Windows administration lab focused on configuring, troubleshooting, and verifying NTFS file and folder permissions using PowerShell and `icacls`.

## 🎯 Objective

Practice investigating and modifying NTFS permissions, testing access with a standard user, troubleshooting access-denied errors, and verifying permission changes.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* NTFS file system
* VMware virtual machine
* Administrator account: `Ginger`
* Test account: `HelpDeskTest`

## 🧩 Scenario

A restricted folder was created to simulate a Help Desk permission issue.

The `HelpDeskTest` account initially had no access to the folder. Permissions were then reviewed and modified by an administrator to provide the required level of access without granting unnecessary Full Control.

## 🔎 Investigation & Procedure

### 01 · Create a Restricted Test Folder

Created a dedicated folder for permission testing.

```powershell
New-Item -Path "C:\SecureLab" -ItemType Directory
```

### 02 · Remove Inherited Permissions

Disabled permission inheritance so the folder could be configured with a controlled ACL.

```powershell
icacls "C:\SecureLab" /inheritance:r
```

### 03 · Configure Administrator and SYSTEM Access

Granted Full Control to the local Administrators group and SYSTEM.

```powershell
icacls "C:\SecureLab" /grant:r "Administrators:(OI)(CI)(F)" "SYSTEM:(OI)(CI)(F)"
```

Verified the resulting permissions:

```powershell
icacls "C:\SecureLab"
```

The folder contained permissions for:

* `SYSTEM` → Full Control
* `Administrators` → Full Control

### 04 · Test Access as a Standard User

Opened a PowerShell session as `HelpDeskTest`.

```powershell
runas /user:HelpDeskTest "powershell.exe"
```

Verified the account:

```powershell
whoami
```

The result confirmed:

```text
windows11vm\helpdesktest
```

Attempted to create a file inside the restricted folder:

```powershell
New-Item "C:\SecureLab\TestFile.txt"
```

The operation returned **Access Denied**, confirming that the standard user did not currently have permission to modify the folder.

### 05 · Verify the ACL as Administrator

Returned to an elevated Administrator PowerShell session and reviewed the folder permissions.

```powershell
icacls "C:\SecureLab"
```

The ACL confirmed that only `SYSTEM` and `Administrators` had access at this stage.

### 06 · Grant the Required Permission

Granted `HelpDeskTest` Modify access instead of Full Control.

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(OI)(CI)(M)"
```

Where:

* `(OI)` = Object Inherit
* `(CI)` = Container Inherit
* `(M)` = Modify

Verified the updated ACL:

```powershell
icacls "C:\SecureLab"
```

The resulting permissions included:

```text
Windows11VM\HelpDeskTest:(OI)(CI)(M)
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
BUILTIN\Administrators:(OI)(CI)(F)
```

### 07 · Verify Standard User Access

Returned to the `HelpDeskTest` PowerShell session and created a file successfully.

```powershell
New-Item "C:\SecureLab\HelpDeskTest.txt"
```

Verified the contents of the folder:

```powershell
Get-ChildItem "C:\SecureLab"
```

The successful file creation confirmed that the required Modify permission was working.

### 08 · Test Permission Boundaries

Attempted to grant Full Control to `HelpDeskTest` from the standard-user session:

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(F)"
```

The operation returned:

```text
Access is denied.
```

This demonstrated that the standard user could modify files within the folder but could not modify the folder's permissions.

### 09 · Verify File Inheritance

Reviewed the permissions inherited by a file inside the folder.

```powershell
(Get-Acl "C:\SecureLab\TestFile.txt").Access |
Format-Table IdentityReference,FileSystemRights,AccessControlType,IsInherited
```

The output confirmed that `HelpDeskTest` inherited Modify permissions from the parent folder.

### 10 · Test File Modification

Verified that the standard user could modify an existing file.

```powershell
Add-Content "C:\SecureLab\TestFile.txt" "Help Desk permission test"
Get-Content "C:\SecureLab\TestFile.txt"
```

The file contents confirmed that the modification was successful.

## 💡 Troubleshooting Considerations

Common NTFS permission issues include:

* Access Denied errors
* Incorrect user permissions
* Inherited permissions
* Missing Modify or Write permissions
* Excessive permissions
* Users attempting administrative actions without elevation
* Permissions applied to the wrong folder or file

Useful commands for investigating NTFS permissions include:

```powershell
Get-Acl "C:\SecureLab"
```

```powershell
icacls "C:\SecureLab"
```

```powershell
(Get-Acl "C:\SecureLab\TestFile.txt").Access
```

## 🔐 Security Notes

* Follow the principle of least privilege.
* Grant users only the permissions required for their role.
* Avoid granting Full Control when Modify access is sufficient.
* Use dedicated test folders when experimenting with permissions.
* Avoid changing permissions on Windows system directories during testing.
* Administrative permission changes should be performed from an elevated session.
* Never publish passwords or other credentials in a repository.

## 📸 Evidence

Screenshots from the lab document:

* Standard user receiving **Access Denied**
* Administrator ACL showing `HelpDeskTest` with Modify access
* Successful file creation and modification
* Permission inheritance verification

> No passwords or credentials should be included in screenshots committed to the repository.

## 📝 What I Learned

* How to investigate NTFS permissions using PowerShell
* How to use `icacls` to configure folder permissions
* How inheritance affects NTFS permissions
* How to troubleshoot Access Denied errors
* How to test permissions using a standard user account
* How to grant Modify access without granting Full Control
* How to verify inherited permissions
* Why administrative changes should be performed from an elevated session
* How the principle of least privilege applies to Windows file permissions
