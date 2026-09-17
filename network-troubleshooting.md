# 🌐 Windows Network Troubleshooting

> A hands-on Windows troubleshooting lab focused on diagnosing common network connectivity issues using PowerShell and Windows networking tools.

## 🎯 Objective

Practice identifying network configuration problems, testing connectivity, troubleshooting communication between hosts, and verifying network functionality using command-line tools.

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* Windows networking tools

## 🧩 Scenario

A Windows workstation is experiencing a network connectivity issue. A Help Desk technician must investigate the workstation's network configuration, test connectivity, identify where communication is failing, and verify the resolution.

## 🔎 Investigation & Procedure

### 01 · Review Network Configuration

Used `ipconfig` to review the workstation's IP configuration.

```powershell
ipconfig
```

Displayed information included:

* IPv4 address
* Subnet mask
* Default gateway
* Network adapter status

For additional configuration details:

```powershell
ipconfig /all
```

### 02 · Test the Local Network Stack

Tested the loopback address to verify that the TCP/IP stack was responding locally.

```powershell
ping 127.0.0.1
```

A successful response confirmed local TCP/IP functionality.

### 03 · Test the Default Gateway

Tested communication with the local network gateway.

```powershell
ping <default-gateway>
```

A successful response indicated that the workstation could communicate with the local gateway.

### 04 · Test Internet Connectivity

Tested connectivity to an external IP address.

```powershell
ping 8.8.8.8
```

This test helped distinguish general network connectivity problems from DNS name-resolution problems.

### 05 · Test Name Resolution

Tested connectivity using a hostname.

```powershell
ping google.com
```

Compared the hostname test with the direct IP-address test to determine whether DNS resolution was functioning.

### 06 · Test the Network Path

Used `tracert` to examine the route between the workstation and a remote destination.

```powershell
tracert google.com
```

The results were reviewed for unreachable hops, excessive delays, or other routing problems.

### 07 · Test a Specific Network Connection

Used PowerShell to test connectivity to a specific host and TCP port.

```powershell
Test-NetConnection google.com -Port 443
```

Reviewed the results for:

* Remote address
* Ping status
* TCP connection status
* Remote port

### 08 · Verify Network Configuration

After troubleshooting, reviewed the network configuration again.

```powershell
ipconfig /all
```

Connectivity tests were repeated to confirm that network communication was functioning correctly.

## 💡 Troubleshooting Considerations

Common Windows network issues include:

* Incorrect IP configuration
* Missing default gateway
* DNS resolution failures
* Network adapter problems
* Connectivity to the local network but not the Internet
* Intermittent connectivity
* Routing problems
* Firewall restrictions
* Incorrect TCP port configuration

Useful troubleshooting commands include:

```powershell
ipconfig
```

```powershell
ipconfig /all
```

```powershell
ping 127.0.0.1
```

```powershell
ping <default-gateway>
```

```powershell
tracert google.com
```

```powershell
Test-NetConnection google.com -Port 443
```

## 🔐 Security Notes

* Do not publish private IP addresses if they reveal information you do not want exposed.
* Review screenshots before committing them to GitHub.
* Avoid publishing VPN addresses, internal hostnames, credentials, or other sensitive network information.
* Network troubleshooting should begin with the least disruptive diagnostic tests.

## 📸 Evidence

Screenshots from the lab document:

* `ipconfig /all` showing network configuration
* Successful local connectivity test
* Default gateway connectivity test
* `tracert` results
* `Test-NetConnection` results
* Final connectivity verification

> Screenshots should not contain passwords, credentials, private information, or other sensitive network details.

## 📝 What I Learned

* How to inspect Windows network configuration
* How to test local and remote connectivity
* How to distinguish IP connectivity from DNS resolution
* How to use `tracert` to investigate network paths
* How to test TCP connectivity with `Test-NetConnection`
* How to approach network troubleshooting systematically
* How to verify connectivity after troubleshooting changes
