# 🖥️ Windows User Account Management

> A hands-on Windows administration lab focused on creating, managing, securing, and troubleshooting local user accounts using PowerShell.

## 🎯 Objective

Practice common Help Desk account-management tasks including creating a local user, managing account state, resetting a password, reviewing account policy, and investigating account lockout behavior.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* Local Windows accounts
* Administrator account: `Ginger`
* Test account: `HelpDeskTest`

## 🧩 Scenario

A dedicated local test account was created to simulate common Help Desk account-management tasks without modifying an existing personal account.

The account was created, added to the local `Users` group, disabled and re-enabled, assigned a new password, and investigated during a controlled account-lockout test.

## 🔎 Investigation & Procedure

### 01 · Create the Test Account

A secure password prompt was used so the password was not entered directly into the command.

```powershell
$Password = Read-Host "Enter a test password" -AsSecureString
```

Created the local account:

```powershell
New-LocalUser -Name "HelpDeskTest" `
    -Password $Password `
    -FullName "Help Desk Test User" `
    -Description "Test account for help desk practice"
```

Added the account to the local `Users` group:

```powershell
Add-LocalGroupMember -Group "Users" -Member "HelpDeskTest"
```

Verified the account:

```powershell
Get-LocalUser -Name "HelpDeskTest"
```

### 02 · Disable and Re-enable the Account

Disabled the test account:

```powershell
Disable-LocalUser -Name "HelpDeskTest"
```

Re-enabled it:

```powershell
Enable-LocalUser -Name "HelpDeskTest"
```

This simulated an administrative account-state change that may be required during Help Desk troubleshooting.

### 03 · Reset the Account Password

A new password was assigned using a secure password prompt:

```powershell
$NewPassword = Read-Host "Enter the new password" -AsSecureString
Set-LocalUser -Name "HelpDeskTest" -Password $NewPassword
```

The password itself was never stored in the documentation or screenshots.

### 04 · Verify Account Status

Reviewed the account's current state:

```powershell
Get-LocalUser -Name "HelpDeskTest" |
Select-Object Name,Enabled,PasswordExpires,LastLogon
```

The account was confirmed to be enabled.

### 05 · Review Local Password and Lockout Policy

Used `net accounts` to review the workstation's local account policy:

```powershell
net accounts
```

The observed configuration included:

| Policy                     | Observed Value |
| -------------------------- | -------------- |
| Maximum password age       | 42 days        |
| Minimum password age       | 0 days         |
| Minimum password length    | 0              |
| Password history           | None           |
| Lockout threshold          | 10 attempts    |
| Lockout duration           | 10 minutes     |
| Lockout observation window | 10 minutes     |

These values represent the local policy configured on the lab workstation at the time of testing.

### 06 · Test Account Lockout

A controlled lockout test was performed using the test account.

The account state was checked with:

```powershell
Get-LocalUser -Name "HelpDeskTest" |
Select-Object Name,Enabled,LockedOut
```

During testing, `Lock
