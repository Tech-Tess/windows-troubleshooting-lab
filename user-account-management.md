# 🖥️ Windows User Account Management

> A hands-on Windows administration lab focused on creating, managing, securing, troubleshooting, and verifying local user accounts using PowerShell.

## 🎯 Objective

Practice creating, managing, verifying, and troubleshooting local Windows user accounts using PowerShell and Windows administrative tools.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* Local Windows user accounts
* VMware virtual machine
* Administrator account: `Ginger`
* Test account: `HelpDeskTest`

## 🧩 Scenario

A dedicated test user account was created to simulate common Help Desk account-management tasks without modifying an existing personal or production account.

The account was created, configured as a standard user, disabled and re-enabled, assigned a new password, and tested against the local account lockout policy.

## 🔎 Investigation & Procedure

### 01 · Create a Secure Test Password

Created a secure password prompt to prevent the password from being exposed in plain text.

```powershell
$Password = Read-Host "Enter a test password" -AsSecureString
```

### 02 · Create a Test User

Created a dedicated local account for Help Desk testing.

```powershell
New-LocalUser -Name "HelpDeskTest" -Password $Password -FullName "Help Desk Test User" -Description "Test account for help desk practice"
```

### 03 · Add the User to the Standard Users Group

Added the test account to the local `Users` group.

```powershell
Add-LocalGroupMember -Group "Users" -Member "HelpDeskTest"
```

This provided standard user access without granting local administrator privileges.

### 04 · Verify the Account

Confirmed that the test account was created and enabled.

```powershell
Get-LocalUser -Name "HelpDeskTest" | Select-Object Name, Enabled, PasswordExpires, LastLogon
```

### 05 · Disable and Re-enable the Account

Simulated a common Help Desk account-status troubleshooting scenario.

```powershell
Disable-LocalUser -Name "HelpDeskTest"
```

Re-enabled the account:

```powershell
Enable-LocalUser -Name "HelpDeskTest"
```

Verified that the account was enabled after the change.

### 06 · Reset the User Password

Reset the account password using a secure password prompt.

```powershell
$NewPassword = Read-Host "Enter the new password" -AsSecureString
Set-LocalUser -Name "HelpDeskTest" -Password $NewPassword
```

### 07 · Review Local Account Policy

Used `net accounts` to review the workstation's password and account lockout policy.

```powershell
net accounts
```

Relevant settings included:

* Maximum password age: 42 days
* Minimum password length: 0
* Password history: None
* Lockout threshold: 10 invalid attempts
* Lockout duration: 10 minutes
* Observation window: 10 minutes

### 08 · Test Account Lockout

Simulated repeated failed authentication attempts and verified the account's lockout state.

```powershell
Get-LocalUser -Name "HelpDeskTest" | Select-Object Name, Enabled, LockedOut
```

The account was observed in a locked state during testing.

After the lockout period expired, the account was checked again and was no longer locked.

### 09 · Verify Final Account State

Confirmed that the test account was enabled and available for continued Help Desk testing.

```powershell
Get-LocalUser -Name "HelpDeskTest" | Select-Object Name, Enabled, LockedOut
```

## 💡 Troubleshooting Considerations

Common local account issues include:

* Disabled accounts
* Incorrect passwords
* Locked accounts
* Missing user accounts
* Insufficient administrative permissions
* Account access problems
* Incorrect account configuration

Useful PowerShell commands for investigation include:

```powershell
Get-LocalUser
```

```powershell
Get-LocalUser -Name "HelpDeskTest" | Select-Object Name, Enabled, LockedOut
```

```powershell
net accounts
```

## 🔐 Security Notes

* Use dedicated test accounts when performing administrative exercises.
* Never publish passwords or credentials in a repository.
* Use secure password prompts instead of storing passwords in plain text.
* Avoid modifying personal or production accounts during testing.
* Standard users should only receive the permissions required for their role.
* Administrative privileges should only be used when required.

## 📸 Evidence

Screenshots from the lab document:

* Test account creation and verification
* Account disable and re-enable testing
* Password and account-status verification
* Local password and lockout policy
* Account lockout testing
* Final account-state verification

> No passwords or credentials should be included in screenshots committed to the repository.

## 📝 What I Learned

* How to create and manage local Windows accounts using PowerShell
* How to add users to local security groups
* How to enable and disable local accounts
* How to securely reset a user's password
* How to review password and account lockout policies
* How to troubleshoot account lockouts
* How to verify account status after administrative changes
* Why dedicated test accounts are useful for Windows administration and Help Desk testing
