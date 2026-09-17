# ⚙️ Windows Services Troubleshooting

> A hands-on Windows administration lab focused on investigating, stopping, starting, and verifying Windows services using PowerShell.

## 🎯 Objective

Practice identifying a Windows service issue, investigating its current state and configuration, performing administrative changes, and verifying that the service is operating correctly.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* Windows Services
* VMware virtual machine
* Administrator account: `Ginger`
* Test service: `Print Spooler`

## 🧩 Scenario

The Print Spooler service was selected to simulate a common Help Desk troubleshooting scenario where a Windows service is stopped and applications depending on that service may not function correctly.

The service was stopped, its configuration was investigated, and it was restarted and verified.

## 🔎 Investigation & Procedure

### 01 · Verify Administrative Access

Confirmed that the PowerShell session was running with administrative privileges.

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

The result was:

```text
True
```

Verified the current account:

```powershell
whoami
```

The result confirmed the Administrator account was being used.

### 02 · Check the Print Spooler Service

Queried the current status of the Print Spooler service.

```powershell
Get-Service -Name Spooler
```

The service was initially running.

### 03 · Simulate a Service Failure

Stopped the Print Spooler service to simulate a service-related Help Desk issue.

```powershell
Stop-Service -Name Spooler
```

Verified the new service state:

```powershell
Get-Service -Name Spooler
```

The service state changed to:

```text
Stopped
```

### 04 · Investigate Service Configuration

Reviewed the service state, startup mode, and service account.

```powershell
Get-CimInstance Win32_Service -Filter "Name='Spooler'" |
Select-Object Name,State,StartMode,StartName
```

The results showed:

* Service: `Spooler`
* State: `Stopped`
* Startup Mode: `Auto`
* Service Account: `LocalSystem`

The `Auto` startup mode indicated that Windows was configured to start the service automatically.

### 05 · Restart the Service

Started the Print Spooler service.

```powershell
Start-Service -Name Spooler
```

Verified that the service returned to a running state:

```powershell
Get-Service -Name Spooler
```

The final state was:

```text
Running
```

## 💡 Troubleshooting Considerations

Common Windows service issues include:

* Service stopped unexpectedly
* Service fails to start
* Incorrect startup configuration
* Dependent services not running
* Insufficient administrative privileges
* Service account or permission problems
* Applications failing because a required service is unavailable

Useful commands for investigating services include:

```powershell
Get-Service
```

```powershell
Get-Service -Name Spooler
```

```powershell
Start-Service -Name Spooler
```

```powershell
Stop-Service -Name Spooler
```

```powershell
Get-CimInstance Win32_Service -Filter "Name='Spooler'" |
Select-Object Name,State,StartMode,StartName
```

## 🔐 Security Notes

* Administrative privileges are required for many service-management operations.
* Do not stop or modify critical Windows services without understanding their purpose.
* Verify the service name before making changes.
* Investigate service dependencies when troubleshooting service failures.
* Use the minimum privileges necessary to perform administrative tasks.

## 📸 Evidence

Screenshots from the lab document:

* Administrative access verification
* Print Spooler service in a stopped state
* Service configuration showing state, startup mode, and service account
* Print Spooler service successfully running after remediation

> Screenshots should not contain passwords, credentials, or other sensitive information.

## 📝 What I Learned

* How to check Windows service status using PowerShell
* How to stop and start a Windows service
* How to investigate service configuration using CIM
* How to identify a service's startup mode
* How to verify the service account being used
* Why administrative privileges are required for service management
* How to verify a service after performing a troubleshooting action
* How service troubleshooting can be approached as a Help Desk workflow
