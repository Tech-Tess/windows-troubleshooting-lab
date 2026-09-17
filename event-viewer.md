# 🔎 Windows Event Viewer Troubleshooting

> A hands-on Windows troubleshooting lab focused on investigating system events, identifying errors and warnings, and using Event Viewer to support Help Desk troubleshooting.

## 🎯 Objective

Practice using Windows Event Viewer to investigate system events, identify relevant errors and warnings, examine event details, and determine useful troubleshooting information.

## 🖥️ Environment

* Windows 11 25H2
* Event Viewer
* PowerShell
* VMware virtual machine

## 🧩 Scenario

A Windows workstation is experiencing an issue that requires investigation. Event Viewer is used to review system and application events and identify information that may help determine the cause of the problem.

The investigation focuses on identifying relevant events, reviewing their details, and documenting findings.

## 🔎 Investigation & Procedure

### 01 · Open Event Viewer

Opened Windows Event Viewer to review the workstation's event logs.

Event Viewer can be launched with:

```powershell
eventvwr.msc
```

### 02 · Review Windows Logs

Reviewed the primary Windows event logs:

* Application
* Security
* System
* Setup

The **System** and **Application** logs were used to investigate system and application-related events.

### 03 · Identify Errors and Warnings

Reviewed recent events and filtered the logs to focus on events with higher severity.

Relevant event levels include:

* Information
* Warning
* Error
* Critical

### 04 · Investigate an Event

Selected a relevant warning or error and reviewed its details.

Information examined included:

* Log name
* Event source
* Event ID
* Level
* Date and time
* User
* Computer
* Event description

### 05 · Review Event Details

Reviewed the **General** and **Details** tabs for the selected event.

The event information was used to determine:

* What component generated the event
* When the event occurred
* What type of issue was reported
* Whether additional investigation was required

### 06 · Investigate Using PowerShell

Used PowerShell to query Windows event logs.

```powershell
Get-WinEvent -LogName System -MaxEvents 20
```

Filtered for error-level events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Level = 2
} -MaxEvents 10
```

### 07 · Compare Event Viewer and PowerShell Results

Compared the events displayed in Event Viewer with the results returned by PowerShell.

This demonstrated how graphical and command-line tools can be used together during Windows troubleshooting.

## 💡 Troubleshooting Considerations

Event Viewer can help investigate:

* Application crashes
* Service failures
* Driver problems
* System errors
* Startup and shutdown issues
* Authentication events
* Hardware-related problems
* Unexpected system behavior

Useful PowerShell commands include:

```powershell
Get-WinEvent -LogName System -MaxEvents 20
```

```powershell
Get-WinEvent -LogName Application -MaxEvents 20
```

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Level = 2
} -MaxEvents 10
```

## 🔐 Security Notes

* Security logs may contain sensitive information.
* Review screenshots before committing them to GitHub.
* Do not publish usernames, credentials, personal information, or other sensitive data.
* Avoid changing or clearing event logs during troubleshooting.
* Preserve relevant event information before making system changes.

## 📸 Evidence

Screenshots from the lab document:

* Event Viewer showing the investigated log
* Relevant warning or error event
* Event details showing Event ID and source
* PowerShell `Get-WinEvent` results

> Screenshots should not contain passwords, credentials, personal information, or other sensitive information.

## 📝 What I Learned

* How to navigate Windows Event Viewer
* How to identify warnings and errors
* How to investigate Event IDs and event sources
* How to review event details
* How to query Windows event logs with PowerShell
* How Event Viewer can support Help Desk troubleshooting
* How to use graphical and command-line tools together during an investigation
