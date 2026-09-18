# ⚡ PowerShell Troubleshooting

> A hands-on Windows administration lab focused on using PowerShell commands, error handling, system information, permissions, service troubleshooting, and verification techniques to investigate and resolve common Windows issues.

## 🎯 Objective

Practice using PowerShell to investigate Windows problems, identify command and permission errors, retrieve system information, verify administrative privileges, troubleshoot NTFS permissions, and confirm troubleshooting results.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell 7
* VMware virtual machine
* Windows administrative tools
* Local administrative account: `Ginger`
* Test account: `HelpDeskTest`

## 🧩 Scenario

A Windows workstation requires troubleshooting using PowerShell. The technician must identify command errors, investigate system information, determine whether administrative privileges are required, troubleshoot a controlled permission issue, and verify Windows service status.

All troubleshooting actions were performed in a controlled VMware lab environment.

## 🔎 Investigation & Procedure

### 01 · Identify Available Commands

Used `Get-Command` to locate PowerShell commands related to Windows services.

```powershell
Get-Command *service*
```

Reviewed the available commands, including:

* `Get-Service`
* `Start-Service`
* `Stop-Service`
* `Restart-Service`
* `Set-Service`
* `New-Service`

This demonstrated how PowerShell can be used to discover available commands before troubleshooting.

### 02 · Get Command Help

Used PowerShell's built-in help system to investigate command syntax, parameters, descriptions, and examples.

```powershell
Get-Help Get-Service
```

PowerShell prompted to update the local help content before displaying the documentation.

Detailed examples were then reviewed with:

```powershell
Get-Help Get-Service -Examples
```

The examples demonstrated different ways to retrieve and work with Windows service objects.

### 03 · Filter Running Services

Executed a service-filtering command to demonstrate PowerShell's pipeline and object filtering capabilities.

```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}
```

The command returned services whose `Status` property was `Running`.

This demonstrated:

* `Get-Service` retrieving service objects
* The pipeline operator `|`
* `Where-Object` filtering objects
* `$_.Status` accessing an object property
* `-eq` performing an equality comparison

### 04 · Test an Invalid Command

Intentionally entered an incorrectly spelled command to observe PowerShell error handling.

```powershell
Get-Servce
```

PowerShell returned a `CommandNotFoundException` because `Get-Servce` is not a recognized cmdlet.

The error included:

```text
CategoryInfo          : ObjectNotFound
FullyQualifiedErrorId : CommandNotFoundException
```

The error was reviewed rather than ignored.

### 05 · Review PowerShell Errors

Used the `$Error` automatic variable to review the most recent PowerShell error.

```powershell
$Error[0]
```

The output returned the previously generated `Get-Servce` error, including the `CommandNotFoundException`.

This demonstrated how PowerShell maintains an error history that can be inspected during troubleshooting.

### 06 · Review System Information

Retrieved selected Windows system information.

```powershell
Get-ComputerInfo |
Select-Object WindowsProductName,WindowsVersion,OsBuildNumber
```

The tested system reported:

```text
WindowsProductName WindowsVersion OsBuildNumber
------------------ -------------- -------------
Windows 10 Pro     2009           26200
```

The product-name field and OS build information returned by PowerShell were recorded as observed during the lab.

### 07 · Verify Administrative Privileges

Checked whether the current PowerShell session had administrative privileges.

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

The Administrator session returned:

```text
True
```

This confirmed that the current PowerShell session was running with elevated privileges.

### 08 · Investigate a Permission Error

Used the `HelpDeskTest` account to reproduce a controlled NTFS permission problem involving `C:\SecureLab`.

The current account was verified with:

```powershell
whoami
```

Result:

```text
windows11vm\helpdesktest
```

An attempt to create a file in the restricted folder failed:

```powershell
New-Item "C:\SecureLab\PermissionTest.txt"
```

PowerShell returned:

```text
Access to the path 'C:\SecureLab\PermissionTest.txt' is denied.
```

The test account was also unable to inspect the folder ACL:

```powershell
icacls "C:\SecureLab"
```

Result:

```text
C:\SecureLab: Access is denied.
Successfully processed 0 files; Failed processing 1 files
```

The permission problem was then investigated from the Administrator session.

### 09 · Inspect and Remediate NTFS Permissions

From the Administrator session, the restricted folder ACL was inspected:

```powershell
icacls "C:\SecureLab"
```

The ACL contained:

```text
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
BUILTIN\Administrators:(OI)(CI)(F)
```

There was no entry granting `HelpDeskTest` access.

The test account was then granted **Modify** permission rather than Full Control:

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(M)"
```

The resulting ACL showed:

```text
Windows11VM\HelpDeskTest:(M)
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
BUILTIN\Administrators:(OI)(CI)(F)
```

The permission assignment was then verified by testing access from the `HelpDeskTest` session.

### 10 · Verify Permission Recovery

A test file was used to verify the permission change.

The existing file initially inherited permissions for SYSTEM and Administrators only. Modify permission was therefore explicitly granted to `HelpDeskTest` on the test file:

```powershell
icacls "C:\SecureLab\PowerShellTest.txt" /grant "HelpDeskTest:(M)"
```

From the `HelpDeskTest` session, the file was successfully modified using:

```powershell
Add-Content "C:\SecureLab\PowerShellTest.txt" "PowerShell permission test"
```

The file was then verified with:

```powershell
Get-Content "C:\SecureLab\PowerShellTest.txt"
```

Result:

```text
PowerShell permission test
```

The current account was also verified with:

```powershell
whoami
```

Result:

```text
windows11vm\helpdesktest
```

This confirmed that the intended test account could access and modify the test file after the permission remediation.

### 11 · Verify Windows Service Status

Used PowerShell to confirm the status and startup configuration of the Print Spooler service.

```powershell
Get-Service -Name Spooler | Select-Object Name, Status, StartType
```

Result:

```text
Name     Status  StartType
----     ------  ---------
Spooler  Running Automatic
```

This confirmed that the Print Spooler service was running and configured for automatic startup.

## 💡 Troubleshooting Considerations

Common PowerShell troubleshooting issues include:

* Command not found
* Incorrect command syntax
* Missing parameters
* Permission errors
* Insufficient administrative privileges
* Incorrect user context
* Unexpected command output
* Windows service issues
* PowerShell errors

Useful troubleshooting commands include:

```powershell
Get-Command
```

```powershell
Get-Help
```

```powershell
$Error[0]
```

```powershell
whoami
```

```powershell
Get-ComputerInfo
```

```powershell
Get-Service
```

```powershell
icacls
```

## 🔐 Security Notes

* Use administrative privileges only when required.
* Do not store passwords or credentials directly in PowerShell scripts.
* Review commands before executing them with elevated privileges.
* Avoid running destructive commands without understanding their effects.
* Use dedicated test accounts and directories for permission testing.
* Apply the minimum permissions required for the task.
* Review screenshots before committing them to GitHub.
* Never publish credentials or sensitive system information in the repository.

## 📸 Evidence

Screenshots from the lab document:

* PowerShell command discovery
* PowerShell help output
* PowerShell help examples
* Running service filtering
* Command error and error investigation
* System information
* Administrative privilege verification
* Permission-related error
* ACL investigation and remediation
* Permission recovery
* Current user verification
* Windows service verification

> Screenshots should not contain passwords, credentials, private information, or unnecessary sensitive system information.

## 📝 What I Learned

* How to discover PowerShell commands
* How to use PowerShell's built-in help system
* How to use the PowerShell pipeline
* How to filter objects with `Where-Object`
* How to identify and investigate `CommandNotFoundException`
* How to use `$Error[0]` to review recent errors
* How to retrieve Windows system information
* How to verify administrative privileges
* How to identify the current user context
* How to troubleshoot NTFS permission problems using PowerShell
* How to inspect and modify ACLs with `icacls`
* How to apply least-privilege permissions
* How to verify that a permission remediation worked
* How to check Windows service status and startup configuration
* How PowerShell can be used throughout the troubleshooting process to investigate, remediate, and verify Windows issues
