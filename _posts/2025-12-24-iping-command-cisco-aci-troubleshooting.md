---
layout: post
title: How to Use the iping Command for Troubleshooting in Cisco ACI
categories: [ACI]
tags: [cisco,network,troubleshooting]
date: 2025-12-24 19:00:00 -0300
---

## Introduction

When troubleshooting connectivity issues in Cisco ACI fabrics, the iping command becomes essential for diagnosing problems beyond the management VRF. The conventional ping command on ACI nodes is restricted to the management VRF, making it impossible to diagnose problems within the fabric’s overlay networks, underlay infrastructure, or customer-specific virtual routing and forwarding domains. This is where `iping`—the fabric-aware ping utility—becomes essential (Strnad, 2022).

The `iping` command extends diagnostic capabilities across the entire ACI fabric by allowing engineers to perform connectivity tests from leaf and spine switches within any VRF context. This technical guide covers everything you need to know about implementing iping for comprehensive fabric troubleshooting.

## Understanding iping vs. Standard Ping

Traditional ICMP ping operates within a single VRF context and cannot traverse the complexities of ACI's overlay and underlay separation. Standard ping on ACI devices is confined to the management VRF, which is insufficient for diagnosing issues involving customer traffic, inter-leaf connectivity, or spine proxy communication (Strnad, 2022; Guo, 2025).

The `iping` command solves this limitation by enabling engineers to specify the VRF context explicitly. This allows testing across the fabric's internal routing domains—including the overlay-1 infrastructure VRF, customer VRFs, and external L3 out networks (Cisco Systems, 2017; Strnad, 2022).

| **Characteristic** | **Standard ping** | **iping** |
|---|---|---|
| VRF Context | Management VRF only | Any VRF (specified via -V flag) |
| Underlay Testing | Not possible | Supported |
| Customer VRF Testing | Not possible | Supported |
| L3 Out Connectivity | Not possible | Supported |
| Execution Context | APIC controllers | Leaf and spine switches |

## Basic Syntax and Structure

The fundamental syntax for iping is straightforward but requires understanding the VRF naming convention specific to ACI (Cisco Systems, 2017):

```bash
iping -V <VRF>:<destination-IP>
```

### Critical VRF Naming Convention

ACI concatenates the tenant name with the VRF name when referencing routing contexts. To identify the correct VRF identifier, first execute:

```bash
show vrf
```

This command displays all available VRFs in the format `[Tenant]:[VRF-Name]`. For example (Sig9 Technical Team, 2017):

```bash
pod1-leaf1# iping -V TENANT-A:VRF-A 192.168.1.1
ICMP echo request sent, id=xxxxx, seq=1 time=1.2 ms
ICMP echo request sent, id=xxxxx, seq=2 time=1.1 ms
```

### The Overlay-1 Exception

One naming inconsistency that confuses many engineers: the `overlay-1` VRF is the fabric's infrastructure VRF, not an actual overlay VRF, despite its name (Guo, 2024). Overlay-1 resides in the underlay network and contains the VTEP addresses for all fabric nodes (APICs, leafs, and spines). IS-IS protocol runs within overlay-1 to maintain infrastructure reachability (Guo, 2024).

To test underlay fabric connectivity between nodes:

```bash
pod1-leaf1# iping -V overlay-1 10.0.59.154
```

## Complete Command Reference

The iping utility supports numerous flags for advanced troubleshooting scenarios (Cisco Systems, 2017).

### Core Parameters

| **Option** | **Purpose** | **Example** |
|---|---|---|
| `-V vrf` | Specify VRF context (REQUIRED) | `-V overlay-1` |
| `host` | Destination IP address or hostname | `10.0.59.154` |

### Packet Control Options

| **Option** | **Description** | **Use Case** |
|---|---|---|
| `-c count` | Number of ping packets to send (default: 5) | `-c 10` sends 10 packets instead of 5 |
| `-s packetsize` | Packet size in bytes | `-s 1500` tests MTU compatibility |
| `-i wait` | Interval between packets (milliseconds) | `-i 100` sends packets 100ms apart |
| `-t timeout` | Timeout for echo reply | `-t 5000` waits 5 seconds for response |

### Source and Routing Options

| **Option** | **Description** | **Use Case** |
|---|---|---|
| `-S source` | Source IP address or interface | `-S 10.0.16.21` pings from specific node |
| `-r` | Disable routing (direct connection only) | `-r` forces direct leaf-to-leaf testing |
| `-F` | Enable do-not-fragment bit | `-F` detects MTU issues precisely |

### Output and Debug Options

| **Option** | **Description** | **Effect** |
|---|---|---|
| `-v` | Verbose output | Displays detailed packet information |
| `-q` | Quiet output | Minimal console feedback |
| `-n` | Numeric output only | Suppresses DNS name resolution |
| `-D` | Enable debug information | Detailed diagnostic output |
| `-d` | Debug mode | Sets SO_DEBUG socket option |

## Practical Usage Examples

### Example 1: Basic Connectivity Test in Overlay-1

Test whether two fabric nodes can reach each other via the underlay (Cisco Systems, 2017):

```bash
pod1-leaf1# iping -V overlay-1 10.0.16.22
64 bytes from 10.0.16.22: icmp_seq=1 ttl=55 time=0.256 ms
64 bytes from 10.0.16.22: icmp_seq=2 ttl=55 time=0.245 ms
64 bytes from 10.0.16.22: icmp_seq=3 ttl=255 time=0.241 ms
64 bytes from 10.0.16.22: icmp_seq=4 ttl=255 time=0.23 ms
64 bytes from 10.0.16.22: icmp_seq=5 ttl=255 time=0.248 ms
--- 10.0.16.22 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.23/0.245/0.256 ms
```

A successful response with zero packet loss indicates healthy underlay connectivity.

### Example 2: Customer VRF Connectivity

Test endpoint reachability within a customer-specific VRF:

```bash
pod1-leaf1# iping -V Customer-Production:VRF-Production 192.168.100.50
64 bytes from 192.168.100.50: icmp_seq=1 ttl=60 time=2.341 ms
64 bytes from 192.168.100.50: icmp_seq=2 ttl=60 time=2.310 ms
64 bytes from 192.168.100.50: icmp_seq=3 ttl=60 time=2.298 ms
64 bytes from 192.168.100.50: icmp_seq=4 ttl=60 time=2.325 ms
64 bytes from 192.168.100.50: icmp_seq=5 ttl=60 time=2.302 ms
--- 192.168.100.50 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.298/2.315/2.341 ms
```

### Example 3: MTU Testing with Large Packets

Detect fragmentation issues by attempting to send packets near the fabric MTU limit (ACI typically supports 9150 bytes on underlay). This approach aligns with network debugging best practices for identifying MTU-related issues (Fasterdata ESnet, 2025):

```bash
pod1-leaf1# iping -V overlay-1 -F -s 9000 10.0.16.22
```

The `-F` flag enables the do-not-fragment bit, causing the router to return an error if the packet exceeds MTU instead of fragmenting it. This reveals MTU misconfigurations between nodes (NetworkWorld, 2021; Fasterdata ESnet, 2025).

### Example 4: Continuous Testing with Custom Intervals

Perform continuous connectivity testing with a longer interval to monitor stability:

```bash
pod1-leaf1# iping -V overlay-1 -c 100 -i 500 10.0.16.22
```

This sends 100 packets with 500-millisecond intervals, useful for detecting intermittent connectivity issues.

### Example 5: Testing from Specific Source Address

Test from a particular node's loopback or interface:

```bash
pod1-leaf1# iping -V overlay-1 -S 10.0.16.21 10.0.16.22
```

### Example 6: Quiet Mode for Scripting

For automation and log parsing, use quiet mode:

```bash
pod1-leaf1# iping -V overlay-1 -q 10.0.16.22
--- 10.0.16.22 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.23/0.245/0.256 ms
```

## Advanced Troubleshooting Scenarios

### Scenario 1: Diagnosing L3Out Connectivity Issues

When external routers connected via L3Out are unreachable, use iping within the corresponding VRF to isolate the problem:

```bash
pod1-leaf1# show vrf
Name                    VniId   State   Reason
black-hole              0       Up      
ext-router:ext-vrf      2228225 Up      
infra                   1       Up      

pod1-leaf1# iping -V ext-router:ext-vrf 203.0.113.1
```

If the external gateway is unreachable via iping, the problem exists between the leaf and external router, not in the ACI overlay.

### Scenario 2: Verifying COOP (Council of Oracles Protocol) Synchronization

To confirm that spine proxy VTEP addresses are reachable from all leaves:

```bash
pod1-leaf1# iping -V overlay-1 10.0.24.65
```

Where `10.0.24.65` is the proxy VTEP address. Failure indicates spine synchronization issues.

### Scenario 3: Testing from Specific Interface

When multiple loopbacks exist, source the ping from a specific interface:

```bash
pod1-leaf1# iping -V overlay-1 -S 10.0.16.21 -c 20 10.0.16.22
```

### Scenario 4: Verbose Debugging for Deep Analysis

Enable verbose output to see detailed packet-level information:

```bash
pod1-leaf1# iping -V overlay-1 -v 10.0.16.22
```

## Integration with Broader Troubleshooting Workflows

iping should be used as part of a comprehensive troubleshooting methodology, not in isolation (Guo, 2025). The typical sequence involves:

1. **APIC Troubleshooting Wizard** (for high-level diagnosis): Launch from Operations > Troubleshooting to identify endpoints and traffic patterns
2. **iping** (for fabric-level verification): Test specific connectivity paths identified by the wizard
3. **show endpoint** (for endpoint tracking): Verify endpoint learning on affected leaves
4. **SPAN/Packet Capture** (for packet analysis): Capture and analyze traffic patterns if iping succeeds but endpoints still fail
5. **Contract Verification**: Ensure network policies permit the desired traffic

This layered approach prevents wasted time on false leads and methodically isolates the failure domain.

## Common Pitfalls and Solutions

**Pitfall 1: Forgetting the `-V` flag**

Attempting `iping 10.0.16.22` without specifying the VRF results in an error or defaults to management VRF, bypassing the intended test.

**Solution**: Always include `-V vrf-name` as a required parameter.

**Pitfall 2: Incorrect VRF Naming**

Using `iping -V Production:VRF-A` when the tenant is actually named `Production-v2` fails silently.

**Solution**: Execute `show vrf` first to confirm exact naming.

**Pitfall 3: Confusing overlay-1 Terminology**

Believing overlay-1 tests VXLAN overlay connectivity when it actually tests underlay (Guo, 2024).

**Solution**: Remember that overlay-1 is the infrastructure VRF in the underlay, not a customer overlay VRF.

**Pitfall 4: Ignoring MTU When Testing Large Deployments**

Attempting large packet sizes without the `-F` flag may succeed locally but fail in production due to MTU restrictions.

**Solution**: Use `-F` to test do-not-fragment scenarios matching real-world conditions.

## Performance Expectations

Under normal fabric conditions, round-trip times for overlay-1 pings typically range from 0.2 to 1 milliseconds between leaf and spine nodes, depending on fabric scale and hardware (Cisco Systems, 2017). Latencies exceeding 10 milliseconds suggest congestion or link issues.

## Conclusion

The `iping` command transforms fabric troubleshooting from a black-box exercise into a methodical, data-driven process. By allowing engineers to test arbitrary IP destinations within specific VRF contexts directly from fabric nodes, iping eliminates a central diagnostic blind spot in ACI deployments. Mastery of iping's flags and options enables rapid identification of connectivity issues, MTU problems, and endpoint reachability failures that would otherwise require extensive log analysis or packet captures.

Incorporate iping into your standard ACI troubleshooting toolkit alongside the APIC GUI tools, and you'll resolve the majority of fabric connectivity issues in minutes rather than hours.

![Alt text for screen readers]({{ "/assets/images/2025/aci-iping-infograph.png"}})


---

## References

Cisco Systems. (2017). *ACI switch command reference, NX-OS release 13.x and later releases* [Technical Documentation]. Cisco Systems, Inc.

Fasterdata ESnet. (2025). *Debugging MTU problems*. Retrieved from https://fasterdata.es.net/network-tuning/mtu-issues/debugging-mtu-problems/

Guo, T. (2024, October 21). ACI curiosities - VRFs overlay-1 and black-hole. *LinkedIn*. Retrieved from https://www.linkedin.com/pulse/aci-curiosities-part-1-luiz-gatof

Guo, T. (2025, September 4). How to troubleshoot ACI connections with Cisco APIC. *LinkedIn*. Retrieved from https://www.linkedin.com/posts/tanzuguo_cisco-aci-troubleshooting-to-troubleshoot-activity-7369722109557710848-6TPy

NetworkWorld. (2021, October 19). *MTU size issues, fragmentation, and jumbo frames*. Retrieved from https://www.networkworld.com/article/745164/mtu-size-issues.html

Sig9 Technical Team. (2017, April 4). Cisco ACI の iping コマンドで Ping を実行する. *Sig9.org*. Retrieved from https://sig9.org/blog/2017/04/04/

Strnad, R. (2022, September 18). How to use iPing on Cisco ACI. *RichardStrnad.ch*. Retrieved from https://www.richardstrnad.ch/posts/iping-cisco-aci/

----

*This content was created with the assistance of AI and carefully reviewed.*

