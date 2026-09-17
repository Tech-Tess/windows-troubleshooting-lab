# ⚡ PowerShell Troubleshooting

> A hands-on Windows administration lab focused on using PowerShell commands, error handling, system information, and troubleshooting techniques to investigate and resolve common Windows issues.

## 🎯 Objective

Practice using PowerShell to investigate Windows problems, identify command and permission errors, retrieve system information, and verify troubleshooting results.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell 7
* VMware virtual machine
* Windows administrative tools

## 🧩 Scenario

A Windows workstation requires troubleshooting using PowerShell. The technician must identify command errors, investigate system information, determine whether administrative privileges are required, and verify the results of troubleshooting actions.

## 🔎 Investigation & Procedure

### 01 · Identify Available Commands

Used `Get-Command` to locate PowerShell commands related to a troubleshooting task.

```powershell
Get-Command *service*
```

Reviewed the available commands and their purposes.

### 02 · Get Command Help

Used PowerShell's built-in help system to investigate command syntax and available parameters.

```powershell
Get-Help Get-Service
```

For detailed examples:

```powershell
Get-Help Get-Service -Examples
```

### 03 · Test an Invalid Command

Intentionally entered an invalid command to observe PowerShell error handling.

```powershell
Get-Servce
```

PowerShell returned an error indicating that the command could not be found.

The error was reviewed rather than ignored.

### 04 · Review PowerShell Errors

Used the `$Error` automatic variable to review recent PowerShell errors.

```powershell
$Error[0]
```

This can help identify the cause of the most recent command failure.

### 05 · Review System Information

Retrieved basic Windows system information.

```powershell
Get-ComputerInfo |
Select-Object WindowsProductName,WindowsVersion,OsBuildNumber
```

The output was reviewed to confirm the operating system and build information.

### 06 · Verify Administrative Privileges

Checked whether the current PowerShell session had administrative privileges.

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

A result of:

```text
True
```

indicates that the current session has administrator privileges.

### 07 · Investigate a Permission Error

Attempted an administrative operation from a standard user session.

```powershell
icacls "C:\SecureLab" /grant "HelpDeskTest:(F)"
```

The operation returned **Access is denied** because the standard user did not have permission to modify the folder's access control list.

The issue demonstrated the importance of verifying the current user's privileges when troubleshooting PowerShell operations.

### 08 · Verify the Current User

Used `whoami` to confirm which account was running the PowerShell session.

```powershell
whoami
```

This is useful when troubleshooting permission-related problems and determining whether the session is running under the expected account.

## 💡 Troubleshooting Considerations

Common PowerShell troubleshooting issues include:

* Command not found
* Incorrect command syntax
* Missing parameters
* Permission errors
* Insufficient administrative privileges
* Incorrect user context
* Unexpected command output
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

## 🔐 Security Notes

* Use administrative privileges only when required.
* Do not store passwords or credentials directly in PowerShell scripts.
* Review commands before executing them with elevated privileges.
* Avoid running destructive commands without understanding their effects.
* Review screenshots before committing them to GitHub.
* Never publish credentials or sensitive system information in the repository.

## 📸 Evidence

Screenshots from the lab document:

* PowerShell command investigation
* PowerShell help output
* Command error and error investigation
* System information
* Administrative privilege verification
* Permission-related error

> Screenshots should not contain passwords, credentials, private information, or other sensitive data.

## 📝 What I Learned

* How to discover PowerShell commands
* How to use PowerShell's built-in help system
* How to investigate PowerShell errors
* How to retrieve Windows system information
* How to verify administrative privileges
* How to identify the current user context
* How PowerShell can be used to troubleshoot Windows permissions
* How to verify troubleshooting results using command-line tools
