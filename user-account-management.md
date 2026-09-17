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

During testing, `LockedOut` was observed as `True`.

After the lockout period expired, the account was checked again and was no longer showing as locked.

## 🧠 Findings

The lab demonstrated a complete local account-management workflow:

1. Create a dedicated test account.
2. Add it to the local `Users` group.
3. Disable and re-enable the account.
4. Reset the password securely.
5. Review local password and lockout policies.
6. Reproduce an account-lockout condition.
7. Verify the account after the lockout condition expired.

The account-management commands were performed against a dedicated test account rather than an existing personal account.

## 🛠️ Troubleshooting Considerations

Common local-account issues that may require investigation include:

* User cannot sign in
* Account is disabled
* Account is locked out
* Password needs to be reset
* Password has expired
* User has incorrect group membership
* Local account policy prevents expected behavior

Useful commands include:

```powershell
Get-LocalUser
```

```powershell
Get-LocalGroupMember -Group "Users"
```

```powershell
Disable-LocalUser
```

```powershell
Enable-LocalUser
```

```powershell
Set-LocalUser
```

```powershell
net accounts
```

## 🔐 Security Notes

* Passwords were entered using secure prompts.
* Passwords were not stored in the repository.
* Account-management testing was performed using a dedicated lab account.
* Administrative privileges should be used only when required.
* Account and password policies should be reviewed carefully before making changes in a production environment.

## 📸 Evidence

Recommended evidence:

1. Test account creation and verification
2. Account disable/re-enable
3. Password/account status verification
4. `net accounts` policy output
5. Account lockout during testing
6. Final account state after the lockout period

Screenshots should not contain passwords or other sensitive information.

## 📝 What I Learned

* How to create and manage local Windows accounts with PowerShell.
* How to enable and disable local accounts.
* How to securely reset a user's password.
* How to review local password and lockout policy.
* How account lockout behavior can be investigated.
* How to document account-management actions and verify their results.
