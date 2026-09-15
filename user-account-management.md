# 🖥️ Windows User Account Management

> A hands-on Windows administration lab focused on creating, managing, verifying, and removing local user accounts.

## 🎯 Objective

Practice creating, managing, verifying, and removing local Windows user accounts using Windows administrative tools and PowerShell.

## 🖥️ Environment

* Windows 11
* PowerShell
* Local Windows user accounts
* Windows administrative tools

## 🧩 Scenario

A test user account is required for a Windows troubleshooting lab. The account will be created, verified, reviewed, and safely removed after testing.

## 🔎 Investigation & Procedure

### 01 · Review Existing User Accounts

Used PowerShell to view the local user accounts currently configured on the system.

```powershell
Get-LocalUser
```

### 02 · Create a Test User

Created a dedicated test account instead of modifying an existing personal account.

```powershell
$Password = Read-Host "Enter a test password" -AsSecureString
New-LocalUser -Name "LabUser" -Password $Password -Description "Windows troubleshooting lab test account"
```

### 03 · Verify the Account

Confirmed that the test account was created successfully.

```powershell
Get-LocalUser -Name "LabUser"
```

### 04 · Review Account Properties

Reviewed the account's status and configuration.

```powershell
Get-LocalUser -Name "LabUser" | Select-Object Name, Enabled, Description, LastLogon
```

### 05 · Remove the Test Account

After completing the exercise, removed the temporary test account.

```powershell
Remove-LocalUser -Name "LabUser"
```

Verified that the account was no longer present:

```powershell
Get-LocalUser
```

## 💡 Troubleshooting Considerations

Common local account issues include:

* Disabled accounts
* Incorrect passwords
* Missing user accounts
* Insufficient administrative permissions
* Account access problems
* Incorrect account configuration

## 🔐 Security Notes

* Use dedicated test accounts when performing administrative exercises.
* Never publish passwords or credentials in a repository.
* Avoid modifying personal or production accounts during testing.
* Administrative privileges should only be used when required.

## 📝 What I Learned

* How to view local Windows accounts using PowerShell
* How to create and remove local test accounts
* How to verify account status and properties
* How to approach common Windows account issues
* Why isolated test accounts are useful for administrative testing
