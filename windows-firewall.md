# 🛡️ Windows Firewall Troubleshooting

> A hands-on Windows administration lab focused on investigating Windows Firewall profiles and rules, testing network access, and verifying firewall configuration using PowerShell.

## 🎯 Objective

Practice reviewing Windows Firewall configuration, investigating firewall rules, testing network connectivity, and understanding how firewall settings can affect Windows network communication.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* Windows Defender Firewall
* VMware virtual machine

## 🧩 Scenario

A Windows workstation is experiencing a network connectivity issue. Windows Defender Firewall is investigated to determine whether firewall configuration or rules could be affecting network communication.

The firewall configuration is reviewed, relevant rules are examined, and connectivity is tested to verify the results.

## 🔎 Investigation & Procedure

### 01 · Review Firewall Profiles

Reviewed the current Windows Firewall profiles.

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction
```

The profiles reviewed included:

* Domain
* Private
* Public

### 02 · Review Firewall Rules

Displayed enabled firewall rules.

```powershell
Get-NetFirewallRule -Enabled True |
Select-Object DisplayName,Direction,Action,Profile |
Format-Table -AutoSize
```

The results were reviewed to identify rules that allow or block network traffic.

### 03 · Search for a Specific Rule

Searched the firewall rules for a specific application or service.

```powershell
Get-NetFirewallRule |
Where-Object DisplayName -Like "*File and Printer Sharing*" |
Select-Object DisplayName,Enabled,Direction,Action
```

This demonstrated how to locate rules associated with a particular Windows networking function.

### 04 · Review Rule Details

Inspected additional information for a firewall rule.

```powershell
Get-NetFirewallRule |
Where-Object DisplayName -Like "*File and Printer Sharing*" |
Get-NetFirewallPortFilter
```

The rule configuration was reviewed to determine which network ports were associated with the rule.

### 05 · Test Network Connectivity

Used PowerShell to test TCP connectivity to a remote service.

```powershell
Test-NetConnection google.com -Port 443
```

Reviewed the result to determine whether the TCP connection was successful.

### 06 · Review Firewall Configuration

Checked the overall firewall configuration again after the investigation.

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction
```

Confirmed that the firewall remained enabled and that the configured default actions were unchanged.

## 💡 Troubleshooting Considerations

Common Windows Firewall issues include:

* Required traffic being blocked
* Incorrect firewall profiles
* Disabled firewall protection
* Incorrect inbound rules
* Incorrect outbound rules
* Application connectivity failures
* Network services being inaccessible
* Rules configured for the wrong profile

Useful troubleshooting commands include:

```powershell
Get-NetFirewallProfile
```

```powershell
Get-NetFirewallRule -Enabled True
```

```powershell
Test-NetConnection google.com -Port 443
```

```powershell
Get-NetFirewallRule |
Where-Object DisplayName -Like "*File and Printer Sharing*"
```

## 🔐 Security Notes

* Avoid disabling Windows Firewall as a first troubleshooting step.
* Review existing rules before creating or modifying firewall rules.
* Use the principle of least privilege when configuring firewall access.
* Avoid creating broad rules when a specific port, application, or profile can be used.
* Do not publish sensitive internal network information in screenshots.
* Review screenshots before committing them to GitHub.

## 📸 Evidence

Screenshots from the lab document:

* Firewall profiles and their enabled state
* Relevant firewall rule investigation
* Firewall rule details
* `Test-NetConnection` results
* Final firewall configuration

> Screenshots should not contain passwords, credentials, private network information, or other sensitive data.

## 📝 What I Learned

* How to review Windows Firewall profiles
* How to investigate Windows Firewall rules using PowerShell
* How firewall profiles affect network traffic
* How to inspect firewall rule properties
* How to test TCP connectivity with `Test-NetConnection`
* How firewall configuration can affect application and network connectivity
* Why firewall changes should be specific and minimally permissive
* How to verify firewall configuration after troubleshooting
