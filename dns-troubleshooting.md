# 🧭 Windows DNS Troubleshooting

> A hands-on Windows troubleshooting lab focused on diagnosing and resolving a controlled DNS configuration failure using PowerShell and Windows networking tools.

## 🎯 Objective

Practice establishing a DNS baseline, reproducing a DNS resolution failure, separating DNS problems from general network connectivity issues, identifying the faulty DNS configuration, restoring the correct configuration, and verifying successful name resolution.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* Windows DNS tools
* Network adapter: `Ethernet0`

## 🧩 Scenario

A controlled DNS failure was introduced on a Windows workstation by changing its configured DNS server to an unreachable private IP address.

A Help Desk technician must determine whether the issue affects general network connectivity or specifically DNS resolution, identify the source of the failure, restore the correct configuration, and verify that hostname resolution is working again.

> This was a controlled troubleshooting exercise performed in a VMware lab environment and was not a production incident.

## 🔎 Investigation & Procedure

### 01 · Establish a DNS Baseline

Reviewed the workstation's network configuration and DNS server settings.

```powershell
ipconfig /all
```

The relevant configuration included:

* IPv4 address: `192.168.125.130`
* Default gateway: `192.168.125.2`
* DHCP server: `192.168.125.254`
* DNS server: `192.168.125.2`
* Network adapter: `Ethernet0`

**📸 Evidence:** `screenshots/dns-ipconfig-baseline.png`

> The screenshot should be reviewed before publication and unnecessary identifiers such as the MAC address should be cropped.

### 02 · Test Baseline DNS Resolution

Used `nslookup` to verify hostname resolution.

```powershell
nslookup google.com
```

The query successfully returned multiple IPv4 and IPv6 addresses.

The configured DNS server appeared as:

```text
Server:  UnKnown
Address:  192.168.125.2
```

`Server: UnKnown` did not indicate a DNS failure. The lookup itself succeeded. The message indicated that the DNS server's hostname could not be resolved through reverse DNS.

**📸 Evidence:** `screenshots/dns-nslookup-baseline.png`

### 03 · Test DNS Resolution with PowerShell

Used `Resolve-DnsName` to perform a DNS lookup.

```powershell
Resolve-DnsName google.com
```

The command successfully returned both `A` and `AAAA` records for `google.com`.

**📸 Evidence:** `screenshots/dns-resolvednsname-baseline.png`

### 04 · Verify the Configured DNS Server

Checked the DNS server configured for `Ethernet0`.

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4
```

The configured DNS server was:

```text
{192.168.125.2}
```

### 05 · Introduce a Controlled DNS Fault

Changed the configured DNS server to an unreachable private IP address.

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses "192.168.125.250"
```

Verified the new configuration:

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4
```

The DNS server was now configured as:

```text
{192.168.125.250}
```

**📸 Evidence:** `screenshots/dns-controlled-fault.png`

### 06 · Reproduce the DNS Failure

Tested hostname resolution after introducing the fault.

```powershell
nslookup google.com
```

The request timed out:

```text
DNS request timed out.
    timeout was 2 seconds.
Server:  UnKnown
Address:  192.168.125.250

*** Request to UnKnown timed-out
```

This reproduced the DNS resolution failure.

**📸 Evidence:** `screenshots/dns-resolution-failure.png`

### 07 · Test General IP Connectivity

Tested direct IP connectivity to determine whether the network connection itself was still functioning.

```powershell
ping 8.8.8.8
```

The test succeeded with:

* 4 packets sent
* 4 packets received
* 0% packet loss
* Approximately 17 ms average latency

This demonstrated that general IP connectivity remained available while DNS resolution was failing.

**📸 Evidence:** `screenshots/dns-ip-connectivity-during-failure.png`

### 08 · Test a Known-Good DNS Server

Sent a DNS query directly to Google's public DNS server.

```powershell
nslookup google.com 8.8.8.8
```

The query succeeded and returned DNS records.

The response identified the DNS server as:

```text
Server:  dns.google
Address:  8.8.8.8
```

The comparison showed:

| Test                                | Result    |
| ----------------------------------- | --------- |
| Configured DNS `192.168.125.250`    | Failed    |
| Direct IP connectivity to `8.8.8.8` | Succeeded |
| DNS query to `8.8.8.8`              | Succeeded |

This isolated the problem to the configured DNS resolver rather than general network connectivity.

**📸 Evidence:** `screenshots/dns-direct-server-test.png`

### 09 · Test Connectivity to the Faulty DNS Server

Tested connectivity to the configured DNS server on TCP port 53.

```powershell
Test-NetConnection 192.168.125.250 -Port 53
```

The test returned:

```text
PingSucceeded     : False
TcpTestSucceeded  : False
```

This provided supporting evidence that the configured DNS server was unreachable.

> TCP port 53 is not the only DNS transport. DNS commonly uses UDP port 53 as well, so the `nslookup` timeout was the primary evidence of the DNS failure.

**📸 Evidence:** `screenshots/dns-server-connectivity-failure.png`

### 10 · Restore the DNS Configuration

Restored the original DNS server configuration.

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses "192.168.125.2"
```

Verified the configuration:

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4
```

The DNS server was restored to:

```text
{192.168.125.2}
```

### 11 · Verify DNS Recovery

Repeated the DNS lookup after restoring the configuration.

```powershell
Resolve-DnsName google.com
```

The command successfully returned multiple `A` and `AAAA` records for `google.com`.

This confirmed that hostname resolution was restored.

**📸 Evidence:** `screenshots/dns-recovery-verification.png`

## 🧠 Troubleshooting Method

The investigation followed a layered troubleshooting process:

```text
DNS resolution failure
        ↓
Verify DNS configuration
        ↓
Reproduce with nslookup
        ↓
Test direct IP connectivity
        ↓
IP connectivity works
        ↓
Test known-good DNS server
        ↓
Known-good DNS works
        ↓
Fault isolated to configured DNS server
        ↓
Restore correct DNS configuration
        ↓
Verify successful DNS resolution
```

This approach helps distinguish DNS-specific failures from broader network connectivity problems.

## 💡 Troubleshooting Considerations

Common DNS issues include:

* Incorrect DNS server configuration
* DNS server unavailable
* DNS resolution failures
* Stale DNS cache
* Incorrect DNS records
* Network connectivity problems
* VPN-related DNS issues
* Firewall restrictions
* Internal hostnames failing to resolve

Useful troubleshooting commands include:

```powershell
ipconfig /all
```

```powershell
nslookup google.com
```

```powershell
Resolve-DnsName google.com
```

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4
```

```powershell
Test-NetConnection 192.168.125.250 -Port 53
```

## 🔐 Security Notes

* Review DNS output before publishing screenshots.
* Avoid exposing internal DNS servers, internal hostnames, or unnecessary private network information.
* Do not publish credentials or other sensitive configuration information.
* DNS troubleshooting should begin with non-destructive diagnostic commands when possible.
* Configuration changes should be documented and restored after controlled testing.

## 📸 Evidence

Screenshots from the lab document:

* DNS baseline configuration
* Baseline `nslookup` results
* Baseline `Resolve-DnsName` results
* Controlled DNS configuration fault
* DNS resolution failure
* Successful IP connectivity during the DNS failure
* Known-good DNS server comparison
* Faulty DNS server connectivity test
* Successful DNS resolution after remediation

> Screenshots should be reviewed before publication to avoid exposing credentials, sensitive information, or unnecessary network identifiers.

## 📝 What I Learned

* How to identify the DNS server configured on a Windows workstation
* How to use `nslookup` to troubleshoot DNS resolution
* How to use `Resolve-DnsName` with PowerShell
* How to distinguish DNS failures from general IP connectivity problems
* How to compare a failing DNS resolver against a known-good DNS server
* How to use `Test-NetConnection` as supporting evidence during network troubleshooting
* How to identify a faulty DNS configuration through layered testing
* How to restore DNS configuration and verify successful resolution
* How controlled fault injection can be used to practice troubleshooting safely
