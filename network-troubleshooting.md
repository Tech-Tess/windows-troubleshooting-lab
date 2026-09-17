# 🌐 Windows Network Troubleshooting

> A hands-on Windows troubleshooting lab demonstrating a structured approach to diagnosing, isolating, and resolving a network connectivity failure using PowerShell and Windows networking tools.

## 🎯 Objective

Practice a Help Desk troubleshooting workflow for a Windows network connectivity issue by:

* Establishing a working network baseline
* Testing connectivity from the local TCP/IP stack to an external destination
* Isolating the point of failure
* Identifying a disabled network adapter as the cause
* Restoring network connectivity
* Verifying the fix with configuration and TCP connectivity tests
* Documenting the troubleshooting process with technical evidence

## 🖥️ Environment

* Windows 11 25H2
* PowerShell
* VMware virtual machine
* Ethernet0 virtual network adapter
* Windows networking tools
* Controlled lab environment

## 🧩 Scenario

A Windows workstation suddenly loses network connectivity.

This exercise intentionally simulated the failure by disabling the VM's `Ethernet0` network adapter. The objective was to investigate the resulting symptoms, determine where connectivity was failing, restore the adapter, and verify successful network communication.

> **Note:** This was a controlled troubleshooting exercise performed inside a VMware virtual machine. It was not a production incident.

---

## 🔎 Investigation & Troubleshooting

### 01 · Establish the Network Baseline

Before introducing the fault, the network adapter status was verified.

```powershell
Enable-NetAdapter -Name "Ethernet0" -Confirm:$false

Get-NetAdapter -Name "Ethernet0" |
    Select-Object Name, Status, LinkSpeed
```

Baseline result:

```text
Name      Status LinkSpeed
----      ------ ---------
Ethernet0 Up     1 Gbps
```

The adapter was operational and reporting a link speed of 1 Gbps before the controlled failure was introduced.

📸 **Evidence:** `screenshots/network-adapter-baseline.png`

---

### 02 · Review the Existing Network Configuration

The VM's network configuration was inspected before troubleshooting.

```powershell
ipconfig /all
```

The VM received its network configuration through DHCP.

Relevant configuration included:

* IPv4 address: `192.168.125.130`
* Subnet mask: `255.255.255.0`
* Default gateway: `192.168.125.2`
* DHCP server: `192.168.125.254`
* DNS server: `192.168.125.2`

The VMware private network addresses above belong to the VM's virtual network and are not the host's public Internet address.

Additional configuration was verified with:

```powershell
Get-NetIPConfiguration
```

📸 **Evidence:** `screenshots/network-ip-configuration.png`

> The original configuration screenshot should be reviewed and cropped before publication so that the virtual adapter's MAC address is not unnecessarily exposed.

---

### 03 · Verify the Local TCP/IP Stack

The loopback address was tested to establish whether the local TCP/IP stack was functioning.

```powershell
ping 127.0.0.1
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

The successful loopback test established that the local TCP/IP stack was responding.

---

### 04 · Verify Local Network Connectivity

The VM's assigned IPv4 address was tested.

```powershell
ping 192.168.125.130
```

The address responded successfully with 0% packet loss.

The default gateway was then tested:

```powershell
ping 192.168.125.2
```

The gateway responded successfully with 0% packet loss.

These tests established that the VM had working local network connectivity before the controlled fault was introduced.

📸 **Evidence:** `screenshots/network-local-connectivity.png`

📸 **Evidence:** `screenshots/network-gateway-connectivity.png`

---

### 05 · Verify External IP Connectivity

An external IP address was tested without relying on DNS name resolution.

```powershell
ping 8.8.8.8
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirmed that the VM could reach an external IP address before the fault was introduced.

📸 **Evidence:** `screenshots/network-external-ip-connectivity.png`

---

### 06 · Verify DNS Resolution and External Connectivity

A hostname was tested to confirm both name resolution and network connectivity.

```powershell
ping google.com
```

The hostname successfully resolved to an external IP address and responded to ICMP requests with 0% packet loss.

This provided evidence that DNS resolution and external network connectivity were functioning during the baseline.

📸 **Evidence:** `screenshots/network-dns-connectivity.png`

---

### 07 · Examine the Network Path

The route to the external destination was inspected using:

```powershell
tracert google.com
```

The first hop was the VMware virtual gateway:

```text
1    <1 ms    <1 ms    <1 ms    192.168.125.2
```

Several intermediate hops returned:

```text
Request timed out.
```

However, the trace ultimately reached the destination successfully.

The presence of timed-out intermediate hops was therefore not treated as proof of an end-to-end routing failure.

📸 **Evidence:** `screenshots/network-traceroute.png`

---

### 08 · Test TCP Connectivity to HTTPS

PowerShell was used to test TCP connectivity to port 443.

```powershell
Test-NetConnection google.com -Port 443
```

The test returned:

```text
RemotePort       : 443
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.125.130
TcpTestSucceeded : True
```

This confirmed that TCP connectivity to the HTTPS service was functioning during the baseline.

📸 **Evidence:** `screenshots/network-tcp-443.png`

---

# 🧪 Controlled Fault Injection

### 09 · Disable the Network Adapter

A controlled network failure was introduced by disabling `Ethernet0`.

```powershell
Disable-NetAdapter -Name "Ethernet0" -Confirm:$false

Get-NetAdapter -Name "Ethernet0" |
    Select-Object Name, Status, LinkSpeed
```

Result:

```text
Name      Status   LinkSpeed
----      ------   ---------
Ethernet0 Disabled 0 bps
```

This created the simulated network outage.

📸 **Evidence:** `screenshots/network-controlled-fault.png`

---

### 10 · Investigate the Adapter Configuration

The IP configuration for the disabled adapter was queried.

```powershell
Get-NetIPConfiguration -InterfaceAlias "Ethernet0"
```

Windows returned an error indicating that no `MSFT_NetIPInterface` object was found for the interface alias.

This was consistent with the adapter being disabled and provided additional evidence that the interface was no longer available as an active IP network interface.

📸 **Evidence:** `screenshots/network-disabled-ipconfig-error.png`

---

### 11 · Test the Default Gateway During the Failure

The default gateway was tested while the adapter remained disabled.

```powershell
ping 192.168.125.2
```

Result:

```text
PING: transmit failed. General failure.
```

All four packets failed, resulting in:

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

The failure occurred before successful communication with the local gateway.

📸 **Evidence:** `screenshots/network-gateway-failure.png`

---

### 12 · Test External Connectivity During the Failure

An external IP address was tested while the adapter remained disabled.

```powershell
ping 8.8.8.8
```

Result:

```text
PING: transmit failed. General failure.
```

The test produced 100% packet loss.

This demonstrated that the problem was not limited to DNS name resolution. The workstation could not transmit network traffic through the disabled adapter.

📸 **Evidence:** `screenshots/network-external-ip-failure.png`

---

# 🧠 Diagnosis

The troubleshooting sequence narrowed the failure to the local network interface.

The evidence showed:

1. The adapter was initially operational.
2. The adapter was intentionally disabled.
3. Windows could no longer provide active IP interface information for `Ethernet0`.
4. Traffic to the local gateway failed with `General failure`.
5. Traffic to an external IP address also failed.
6. The failure occurred without relying on DNS resolution.

### Diagnosis

**The simulated connectivity failure was caused by the `Ethernet0` network adapter being disabled.**

The troubleshooting process demonstrated how testing from the local stack outward can help isolate the layer where connectivity is failing.

---

# 🔧 Remediation

The network adapter was re-enabled.

```powershell
Enable-NetAdapter -Name "Ethernet0" -Confirm:$false

Get-NetAdapter -Name "Ethernet0" |
    Select-Object Name, Status, LinkSpeed
```

Result:

```text
Name      Status LinkSpeed
----      ------ ---------
Ethernet0 Up     1 Gbps
```

The adapter returned to an operational state.

---

# ✅ Verification

The restored IP configuration was verified:

```powershell
Get-NetIPConfiguration -InterfaceAlias "Ethernet0"
```

The VM had regained:

* IPv4 address: `192.168.125.130`
* Default gateway: `192.168.125.2`
* DNS server: `192.168.125.2`

Final TCP connectivity was then tested:

```powershell
Test-NetConnection google.com -Port 443
```

Result:

```text
ComputerName     : google.com
RemotePort       : 443
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.125.130
TcpTestSucceeded : True
```

The successful TCP test confirmed that network connectivity had been restored.

📸 **Evidence:** `screenshots/network-recovery-verification.png`

---

# 🧭 Troubleshooting Method

This exercise followed a structured troubleshooting sequence:

```text
Establish baseline
       ↓
Identify symptoms
       ↓
Test local connectivity
       ↓
Test gateway
       ↓
Test external connectivity
       ↓
Inspect adapter state
       ↓
Identify disabled adapter
       ↓
Restore adapter
       ↓
Verify IP configuration
       ↓
Verify TCP connectivity
```

This approach helps avoid making assumptions based on a single failed test and instead uses multiple observations to isolate the failure.

---

## 💡 Troubleshooting Considerations

Common Windows network problems can include:

* Disabled or disconnected network adapters
* Incorrect IP configuration
* Missing or incorrect default gateway
* DNS resolution failures
* DHCP problems
* Routing issues
* Firewall restrictions
* TCP port connectivity problems
* Intermittent network connectivity

Useful diagnostic commands include:

```powershell
ipconfig /all
```

```powershell
Get-NetAdapter
```

```powershell
Get-NetIPConfiguration
```

```powershell
ping 127.0.0.1
```

```powershell
ping <default-gateway>
```

```powershell
ping 8.8.8.8
```

```powershell
ping google.com
```

```powershell
tracert google.com
```

```powershell
Test-NetConnection google.com -Port 443
```

---

## 🔐 Security Notes

* Review screenshots before publishing them to GitHub.
* Avoid publishing credentials, passwords, tokens, VPN information, or sensitive internal hostnames.
* Review network configuration screenshots for MAC addresses and other information that does not need to be public.
* Use controlled test environments when intentionally disrupting network connectivity.
* Prefer the least disruptive diagnostic test that can provide useful information.
* Restore any intentionally modified network configuration after testing.

---

## 📸 Evidence

Selected screenshots from the controlled troubleshooting exercise:

| Evidence                      | Screenshot                                        |
| ----------------------------- | ------------------------------------------------- |
| Network adapter baseline      | `screenshots/network-adapter-baseline.png`        |
| Network configuration         | `screenshots/network-ip-configuration.png`        |
| Local connectivity            | `screenshots/network-local-connectivity.png`      |
| Gateway connectivity          | `screenshots/network-gateway-connectivity.png`    |
| Traceroute                    | `screenshots/network-traceroute.png`              |
| TCP 443 baseline              | `screenshots/network-tcp-443.png`                 |
| Controlled adapter failure    | `screenshots/network-controlled-fault.png`        |
| IP configuration failure      | `screenshots/network-disabled-ipconfig-error.png` |
| Gateway failure               | `screenshots/network-gateway-failure.png`         |
| External connectivity failure | `screenshots/network-external-ip-failure.png`     |
| Recovery verification         | `screenshots/network-recovery-verification.png`   |

> Screenshots should be reviewed before publication to remove credentials, sensitive information, and unnecessary network identifiers.

---

## 📝 What I Learned

* How to establish a network baseline before troubleshooting.
* How to test connectivity progressively from the local TCP/IP stack to an external destination.
* How to distinguish an IP connectivity problem from a DNS-specific problem.
* How to use `Get-NetAdapter` and `Get-NetIPConfiguration` to investigate Windows network interfaces.
* How a disabled network adapter can prevent communication with both the local gateway and external destinations.
* How to use `tracert` to examine a network path without assuming that every timed-out intermediate hop represents a failure.
* How to use `Test-NetConnection` to verify TCP connectivity to a specific port.
* How to validate a remediation rather than stopping after making a configuration change.
* How to document a troubleshooting process using observable evidence and a clear diagnosis.
