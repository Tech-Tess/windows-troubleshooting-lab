# 🔐 Windows NTFS Permissions Troubleshooting

> A hands-on Windows administration lab focused on investigating NTFS permissions, reproducing access-denied conditions, applying least-privilege access, and verifying the resulting ACLs.

## 🎯 Objective

Practice investigating and modifying Windows NTFS permissions using PowerShell and `icacls`, while understanding how permissions affect a standard user.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* NTFS file system
* Administrator account: `Ginger`
* Test account: `HelpDeskTest`

## 🧩 Scenario

A restricted folder was created to simulate a user reporting that they could not access a required resource.

The `HelpDeskTest` account was unable to access the restricted folder. The permissions were investigated, the existing ACL was reviewed, Modify access was granted by an administrator, and access was verified.

## 🔎 Investigation & Procedure

### 01 · Create a Restricted Folder

Created a dedicated lab folder:

```powershell
New-Item -Path "C:\SecureLab" -ItemType Directory
```

Inheritance was removed:

```powershell
icacls "C:\SecureLab" /inheritance:r
```

Full Control was granted to the required administrative principals:

```powershell
icacls "C:\SecureLab" /grant:r `
    "Administrators:(OI)(CI)(F)" `
    "SYSTEM:(OI)(CI)(F)"
```

Verified the ACL:

```powershell
icacls "C:\SecureLab"
```

The resulting ACL contained Full Control for:

* `BUILTIN\Administrators`
* `NT AUTHORITY\SYSTEM`

### 02 · Test Access as the Standard User

A PowerShell session was launched as `HelpDeskTest`:

```powershell
runas /user:HelpDeskTest "powershell.exe"
```

The account was verified:

```powershell
whoami
```

The result confirmed:

```text
windows11vm\helpdesktest
```

The user then attempted to create a file:

```powershell
New-Item "C:\SecureLab\TestFile.txt"
```

The operation returned **Access is denied**.

This reproduced the permission problem in a controlled environment.

### 03 · Investigate the ACL

The administrator account reviewed the folder permissions:

```powershell
icacls "C:\SecureLab"
```

The ACL did not contain an entry granting `HelpDeskTest` access.

### 04 · Apply Least-Privilege Access

Rather than granting Full Control, Modify access was assigned to the test account:

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(OI)(CI)(M)"
```

The ACL was verified:

```powershell
icacls "C:\SecureLab"
```

The resulting permissions included:

```text
Windows11VM\HelpDeskTest:(OI)(CI)(M)
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
BUILTIN\Administrators:(OI)(CI)(F)
```

`(M)` represents Modify permission.

`(OI)(CI)` specifies that the permission is inherited by files and subfolders within the folder.

### 05 · Verify File Access

The `HelpDeskTest` account successfully created and accessed a file:

```powershell
New-Item "C:\SecureLab\HelpDeskTest.txt"
```

The folder contents were verified:

```powershell
Get-ChildItem "C:\SecureLab"
```

The test user was also able to modify an existing file:

```powershell
Add-Content "C:\SecureLab\TestFile.txt" "Help Desk permission test"
```

The file contents were verified:

```powershell
Get-Content "C:\SecureLab\TestFile.txt"
```

### 06 · Test Permission Boundaries

The standard user attempted to grant itself Full Control:

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(F)"
```

The operation returned **Access is denied**.

This demonstrated that the standard user could use the permissions granted to it without being able to independently elevate its own access.

### 07 · Verify Permission Inheritance

The permissions inherited by a file were reviewed:

```powershell
(Get-Acl "C:\SecureLab\TestFile.txt").Access |
Format-Table IdentityReference,FileSystemRights,AccessControlType,IsInherited
```

The results showed inherited permissions for the folder's configured principals.

## 🧠 Findings

The troubleshooting workflow demonstrated:

1. Reproduce an Access Denied condition.
2. Identify the affected user.
3. Inspect the NTFS ACL.
4. Determine which permissions were missing.
5. Apply the minimum required access.
6. Verify successful file access.
7. Test that the user could not grant itself additional privileges.

The final configuration gave `HelpDeskTest` Modify access while retaining Full Control for SYSTEM and Administrators.

## 🛠️ Troubleshooting Considerations

When troubleshooting Windows access problems, investigate:

* NTFS permissions
* Inherited permissions
* Explicit permissions
* User and group membership
* Ownership
* Share permissions when network shares are involved
* Whether the user is actually using the expected account

Useful commands include:

```powershell
Get-Acl "C:\SecureLab"
```

```powershell
icacls "C:\SecureLab"
```

```powershell
whoami
```

## 🔐 Security Notes

* Least privilege was used instead of granting unnecessary Full Control.
* Testing was performed on a dedicated lab folder.
* Broad system directories were not modified.
* Administrative access was used only for permission-management tasks.
* NTFS permissions should be reviewed carefully before changing production resources.

## 📸 Evidence

Recommended evidence:

1. `Access is denied` when `HelpDeskTest` initially attempted to access the restricted folder.
2. Administrator ACL showing `HelpDeskTest` with Modify access.
3. Successful file creation/modification by `HelpDeskTest`.
4. Optional permission-inheritance verification.

Screenshots should not contain passwords or unnecessary personal information.

## 📝 What I Learned

* How Windows NTFS permissions are represented.
* How to inspect ACLs using `Get-Acl` and `icacls`.
* How inherited permissions affect files and folders.
* How to reproduce and investigate Access Denied errors.
* How to apply Modify permissions using `icacls`.
* Why least privilege is preferable to unnecessary Full Control.
* How to verify that a permission change actually solved the access problem.
