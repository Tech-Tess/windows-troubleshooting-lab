# 🛡️ Windows Firewall Troubleshooting

> A hands-on Windows administration lab focused on investigating Windows Defender Firewall configuration, testing network access, analyzing firewall rules, and verifying remediation using PowerShell.

## 🎯 Objective

Practice reviewing Windows Firewall configuration, testing network connectivity, creating and investigating a controlled firewall rule, identifying its effect on network traffic, removing the rule, and verifying successful recovery.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* Windows Defender Firewall
* VMware virtual machine
* Administrator PowerShell session

## 🧩 Scenario

A controlled firewall fault was introduced to simulate a connectivity issue.

Baseline firewall configuration and TCP connectivity were established first. A narrowly scoped outbound firewall rule was then created to block TCP traffic to a specific remote IP address on port 443.

Connectivity was tested during the fault, the firewall rule was investigated, the rule was removed, and connectivity was verified again.

> This was a controlled lab exercise and not a production incident.

## 🔎 Investigation & Procedure

### 01 · Review Firewall Profiles

Reviewed the current Windows Firewall profiles.

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction
```

All three firewall profiles were enabled:

* Domain: Enabled
* Private: Enabled
* Public: Enabled

The default inbound and outbound actions were reported as `NotConfigured`.

📸 **Evidence:** `screenshots/firewall-profile-baseline.png`

### 02 · Establish Baseline TCP Connectivity

Tested HTTPS connectivity to Google over TCP port 443.

```powershell
Test-NetConnection google.com -Port 443
```

The connection succeeded:

```text
ComputerName       : google.com
RemoteAddress      : 64.233.178.101
RemotePort         : 443
InterfaceAlias     : Ethernet0
SourceAddress      : 192.168.125.130
TcpTestSucceeded   : True
```

This established that TCP 443 connectivity was working before introducing the controlled firewall fault.

📸 **Evidence:** `screenshots/firewall-connectivity-baseline.png`

### 03 · Create a Controlled Firewall Block Rule

Created a temporary outbound firewall rule targeting the resolved remote IP address and TCP port 443.

```powershell
$TestIP = "64.233.178.101"

New-NetFirewallRule `
  -DisplayName "HelpDeskLab-Block-Test443" `
  -Direction Outbound `
  -Action Block `
  -Protocol TCP `
  -RemoteAddress $TestIP `
  -RemotePort 443
```

The rule was created successfully with:

* Enabled: `True`
* Direction: `Outbound`
* Action: `Block`
* Profile: `Any`
* Status: `OK`

📸 **Evidence:** `screenshots/firewall-controlled-block.png`

### 04 · Test Connectivity During the Fault

Retested TCP connectivity to the same remote IP and port.

```powershell
```
