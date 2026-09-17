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
Test-NetConnection $TestIP -Port 443
```

The result showed:

```text
PingSucceeded      : True
TcpTestSucceeded   : False
```

This demonstrated that the remote host remained reachable by ICMP while the TCP connection to port 443 failed.

The result was consistent with the controlled outbound firewall rule.

📸 **Evidence:** `screenshots/firewall-blocked-connectivity.png`

### 05 · Investigate the Firewall Rule

Verified the configuration of the temporary firewall rule.

```powershell
Get-NetFirewallRule -DisplayName "HelpDeskLab-Block-Test443" |
Select-Object DisplayName,Enabled,Direction,Action,Profile
```

Output confirmed:

```text
DisplayName : HelpDeskLab-Block-Test443
Enabled     : True
Direction   : Outbound
Action      : Block
Profile     : Any
```

The configured remote address was then verified:

```powershell
Get-NetFirewallRule -DisplayName "HelpDeskLab-Block-Test443" |
Get-NetFirewallAddressFilter
```

Output:

```text
LocalAddress  : Any
RemoteAddress : 64.233.178.101
```

The rule therefore applied to outbound traffic from the VM to the specified remote IP address.

📸 **Evidence:** `screenshots/firewall-rule-investigation.png`

### 06 · Remove the Blocking Rule

Removed the temporary firewall rule to remediate the controlled fault.

```powershell
Remove-NetFirewallRule -DisplayName "HelpDeskLab-Block-Test443"
```

No output was returned, indicating the command completed successfully.

### 07 · Verify Connectivity Recovery

Retested the same TCP connection.

```powershell
Test-NetConnection $TestIP -Port 443
```

The connection succeeded:

```text
ComputerName     : 64.233.178.101
RemoteAddress    : 64.233.178.101
RemotePort       : 443
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.125.130
TcpTestSucceeded : True
```

This verified that removing the blocking rule restored TCP connectivity.

📸 **Evidence:** `screenshots/firewall-recovery-verification.png`

### 08 · Verify the Temporary Rule Was Removed

Confirmed that the test rule no longer existed.

```powershell
Get-NetFirewallRule -DisplayName "HelpDeskLab-Block-Test443" -ErrorAction SilentlyContinue
```

No output was returned.

This confirmed that the temporary firewall rule had been removed.

### 09 · Verify Final Firewall Configuration

Reviewed the firewall profiles again after remediation.

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction
```

Final results matched the baseline:

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True        NotConfigured         NotConfigured
Private    True        NotConfigured         NotConfigured
Public     True        NotConfigured         NotConfigured
```

The firewall remained enabled after the troubleshooting exercise.

📸 **Evidence:** `screenshots/firewall-final-profile.png`

## 🧠 Troubleshooting Analysis

The troubleshooting process followed a layered approach:

```text
Baseline connectivity
        ↓
Introduce controlled firewall rule
        ↓
TCP 443 connection fails
        ↓
Ping remains successful
        ↓
Investigate firewall rule
        ↓
Confirm outbound block to target IP:443
        ↓
Remove rule
        ↓
TCP 443 connection succeeds
        ↓
Verify final firewall configuration
```

The controlled test demonstrated that firewall rules can affect specific network traffic without necessarily preventing all communication with a remote host.

In this exercise, ICMP connectivity remained successful while the TCP connection to port 443 was blocked.

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
* Rules targeting the wrong application, port, protocol, or address

Useful commands include:

```powershell
Get-NetFirewallProfile
```

```powershell
Get-NetFirewallRule -Enabled True
```

```powershell
Test-NetConnection <host> -Port <port>
```

```powershell
Get-NetFirewallRule -DisplayName "<rule name>"
```

```powershell
Get-NetFirewallRule -DisplayName "<rule name>" |
Get-NetFirewallAddressFilter
```

## 🔐 Security Notes

* Avoid disabling Windows Firewall as a first troubleshooting step.
* Review existing rules before creating or modifying firewall rules.
* Use the principle of least privilege when configuring firewall access.
* Prefer narrowly scoped rules using specific ports, protocols, applications, profiles, or addresses where appropriate.
* Remove temporary troubleshooting rules after testing.
* Avoid creating broad firewall rules when a more specific rule is sufficient.
* Review screenshots before committing them to GitHub.
* Do not publish passwords, credentials, or unnecessary sensitive network information.

## 📸 Evidence

Key screenshots from the lab:

* `screenshots/firewall-profile-baseline.png`
* `screenshots/firewall-connectivity-baseline.png`
* `screenshots/firewall-controlled-block.png`
* `screenshots/firewall-blocked-connectivity.png`
* `screenshots/firewall-rule-investigation.png`
* `screenshots/firewall-recovery-verification.png`
* `screenshots/firewall-final-profile.png`

> Screenshots were reviewed before publication to avoid exposing credentials or unnecessary sensitive information.

## 📝 What I Learned

* How Windows Defender Firewall operates as a built-in Windows security control.
* How to review Windows Firewall profiles using PowerShell.
* How firewall rules determine whether specific network traffic is allowed or blocked.
* How to create a narrowly scoped firewall rule for controlled troubleshooting.
* How to investigate firewall rules and their configured addresses.
* How to distinguish general network reachability from TCP service connectivity.
* How to verify whether a firewall rule is affecting a specific connection.
* How to safely remove a temporary firewall rule after testing.
* Why firewall changes should be specific, minimally permissive, and verified after remediation.
