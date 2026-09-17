# 🧭 Windows DNS Troubleshooting

> A hands-on Windows troubleshooting lab focused on diagnosing DNS resolution issues using PowerShell and Windows networking tools.

## 🎯 Objective

Practice investigating DNS configuration, testing hostname resolution, identifying DNS-related failures, and verifying successful name resolution.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* Windows DNS tools

## 🧩 Scenario

A Windows workstation can communicate with network resources by IP address but is experiencing problems accessing resources by hostname.

A Help Desk technician must determine whether DNS name resolution is functioning correctly and identify the source of the problem.

## 🔎 Investigation & Procedure

### 01 · Review Network Configuration

Reviewed the workstation's network configuration and DNS server settings.

```powershell
ipconfig /all
```

The output was reviewed for:

* IPv4 address
* Default gateway
* DNS servers
* Network adapter status

### 02 · Test DNS Resolution

Used `nslookup` to test hostname resolution.

```powershell
nslookup google.com
```

Reviewed the response for:

* DNS server used
* Resolved IP address
* DNS response status

### 03 · Test DNS Resolution with PowerShell

Used `Resolve-DnsName` to perform a DNS lookup.

```powershell
Resolve-DnsName google.com
```

Reviewed the returned records and DNS server information.

### 04 · Compare IP Connectivity and DNS Resolution

Tested direct IP connectivity separately from hostname resolution.

```powershell
ping 8.8.8.8
```

Then tested the hostname:

```powershell
ping google.com
```

Comparing these results helps determine whether a connectivity problem is related to DNS resolution or general network connectivity.

### 05 · Review the DNS Client Cache

Displayed cached DNS records on the workstation.

```powershell
ipconfig /displaydns
```

The cache was reviewed for previously resolved hostnames.

### 06 · Clear the DNS Resolver Cache

Cleared the local DNS resolver cache to remove potentially outdated cached records.

```powershell
ipconfig /flushdns
```

The command was used as a troubleshooting step before testing resolution again.

### 07 · Verify DNS Resolution

Repeated the DNS lookup after clearing the cache.

```powershell
nslookup google.com
```

Confirmed that hostname resolution was functioning after troubleshooting.

## 💡 Troubleshooting Considerations

Common DNS issues include:

* Incorrect DNS server configuration
* DNS server unavailable
* DNS resolution failures
* Stale DNS cache
* Incorrect DNS records
* Network connectivity problems
* VPN-related DNS issues
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
ipconfig /displaydns
```

```powershell
ipconfig /flushdns
```

## 🔐 Security Notes

* Review DNS output before publishing screenshots.
* Avoid exposing internal DNS servers, internal hostnames, or private network information unnecessarily.
* Do not publish credentials or other sensitive configuration information.
* DNS troubleshooting should begin with non-destructive diagnostic commands.

## 📸 Evidence

Screenshots from the lab document:

* DNS server configuration from `ipconfig /all`
* `nslookup` results
* `Resolve-DnsName` results
* DNS cache information
* Successful DNS resolution after troubleshooting

> Screenshots should not contain passwords, credentials, private network information, or other sensitive data.

## 📝 What I Learned

* How DNS resolution works from a Windows workstation
* How to identify configured DNS servers
* How to use `nslookup` for DNS troubleshooting
* How to use `Resolve-DnsName` with PowerShell
* How to distinguish DNS problems from general connectivity problems
* How to inspect and clear the Windows DNS resolver cache
* How to verify DNS resolution after troubleshooting
