# ⚙️ Windows Services Troubleshooting

> A hands-on Windows administration lab focused on investigating, managing, and verifying Windows services using PowerShell.

## 🎯 Objective

Practice identifying a Windows service availability issue, investigating its configuration, performing an administrative remediation, and verifying that the service has returned to the expected state.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* Windows Services
* VMware virtual machine
* Administrator account: `Ginger`
* Test service: `Print Spooler`

## 🧩 Scenario

The Print Spooler service was intentionally stopped to simulate a workstation where printing functionality was unavailable.

The service was investigated using PowerShell, its current configuration was reviewed, the service was restored, and its final state was verified.

This was a controlled troubleshooting exercise performed in a Windows 11 virtual machine.

## 🔎 Investigation & Procedure

### 01 · Verify Administrative Access

Before modifying the service, confirmed that the PowerShell session was running with administrative privileges.

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

The result confirmed the `Ginger` administrator account was being used.

### 02 · Check the Print Spooler Service

Queried the current state of the Print Spooler service:

```powershell
Get-Service -Name Spooler
```

The service was initially in the:

```text
Running
```

state.

### 03 · Simulate a Service Failure

Stopped the Print Spooler service to create the controlled troubleshooting condition:

```powershell
Stop-Service -Name Spooler
```

Verified the service state:

```powershell
Get-Service -Name Spooler
```

The service state changed to:

```text
Stopped
```

This reproduced the condition of a required Windows service being unavailable.

### 04 · Investigate Service Configuration

Reviewed the service state, startup mode, and service account:

```powershell
Get-CimInstance Win32_Service -Filter "Name='Spooler'" |
Select-Object Name,State,StartMode,StartName
```

The results showed:

| Property        | Result        |
| --------------- | ------------- |
| Service         | `Spooler`     |
| State           | `Stopped`     |
| Startup Mode    | `Auto`        |
| Service Account | `LocalSystem` |

The `Auto` startup configuration indicates that Windows is configured to start the service automatically.

### 05 · Restore the Service

Started the Print Spooler service:

```powershell
Start-Service -Name Spooler
```

### 06 · Verify Service Recovery

Confirmed that the Print Spooler service returned to a running state:

```powershell
Get-Service -Name Spooler
```

The final state was:

```text
Running
```

The service was successfully restored after the controlled failure was introduced.

## 🧠 Findings

The investigation demonstrated the following troubleshooting workflow:

1. Confirm administrative privileges.
2. Establish the service's normal state.
3. Reproduce a controlled service-availability problem.
4. Verify that the service is actually stopped.
5. Investigate the service configuration.
6. Restore the service.
7. Verify the final operational state.

The Print Spooler service was configured for automatic startup and ran under the `LocalSystem` service account.

After being intentionally stopped, the service was successfully restarted and verified as `Running`.

## 🛠️ Troubleshooting Considerations

When a Windows application or feature is unavailable, the underlying Windows service should be investigated before assuming the application itself is defective.

Common service-related issues include:

* Service stopped unexpectedly
* Service fails to start
* Incorrect startup configuration
* Service dependencies not running
* Insufficient administrative privileges
* Service account or permission problems
* Applications failing because a required service is unavailable

Useful commands include:

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
* Perform service-management testing in a dedicated lab environment when possible.

## 📸 Evidence

The following screenshots document the troubleshooting process:

1. `services-admin-verification.png`

   * PowerShell session confirming administrative privileges.

2. `services-spooler-stopped.png`

   * Print Spooler showing a `Stopped` state during the controlled test.

3. `services-spooler-configuration.png`

   * Service configuration showing the service state, startup mode, and service account.

4. `services-spooler-running.png`

   * Print Spooler showing a `Running` state after remediation.

> Screenshots should not contain passwords, credentials, or other sensitive information.

## 📝 What I Learned

* How to check Windows service status using PowerShell.
* How to stop and start a Windows service.
* How to investigate service configuration using CIM.
* How to identify a service's startup mode.
* How to identify the account under which a service runs.
* Why administrative privileges are required for service management.
* How to verify a service after performing a troubleshooting action.
* How to use a controlled failure to practice a repeatable Help Desk troubleshooting workflow.
