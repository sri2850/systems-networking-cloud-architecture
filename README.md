---
### Enterprise Systems, Networking & Cloud Architecture Track
---
#### **Phase 1: Process, CPU, Memory Internals & Core Shell Tooling**
---
1. **Process Lifecycles & Signals:** States (`R`, `S`, `D`, `Z`); defunct & orphan processes; why `kill -9` fails on zombies; reaping via PID 1 (`systemd`).
2. **File Descriptors & Inode Mechanics:** Virtual filesystem tables (`/proc/<PID>/fd/`); resolving 100% ghost disk leaks from deleted-but-open files via truncation (`: > /proc/<PID>/fd/<FD>`).
3. **Memory Sizing & Paging Mechanics:** VSS (`VmSize`) vs RSS (`VmRSS`) vs PSS (`/proc/<PID>/smaps_rollup`); page cache, dirty pages writeback daemon (`kswapd`/`flush`), and swap thrashing diagnostics.
4. **The Out-Of-Memory (OOM) Subsystem:** Memory overcommit settings (`/proc/sys/vm/overcommit_memory`); kernel badness scoring (`oom_score`, `oom_score_adj`); kernel ring buffer forensics via `dmesg -T`.
5. **CPU Scheduling & Queue Depth Mechanics:** Load Average as total queue depth (`R` + `D` tasks); high compute saturation vs storage wait; `vmstat 1` triage matrix (`r`, `b`, `us`, `wa`, `so`, `si`).
6. **Core Shell Tooling & Resilient Scripting:** Text parsing primitives (`grep -E`, `awk`, `sed`, `find`, `xargs`); production bash standards (`set -euo pipefail`, trap handlers, `&> /dev/null`).
---
---
#### **Phase 2: Storage Subsystems, Filesystems, LVM & OverlayFS**
---
7. **Block Devices, Superblocks & Inode Mechanics:** Inode table structure vs raw storage data blocks; inode starvation (`df -i` at 100%, `df -h` empty) throwing `No space left on device`; streaming deletion via `find /path -type f -delete` to avoid shell `ARG_MAX` buffer overflows.
8. **Logical Volume Manager (LVM) Hierarchy:** 3-tier mapping (`pvcreate` -> `vgcreate` -> `lvcreate`); online, zero-downtime block growth via `lvextend -r`.
9. **Filesystem Internals & Resilience:** `ext4` vs `XFS` operational differences (one-way expansion vs shrinking limitations); filesystem repair tools (`fsck`, `xfs_repair`) and journal verification.
10. **Storage Mounting & Persistence:** Persistent UUID mounting (`/etc/fstab`) to avoid device node letter swapping (`/dev/sdX`); preventing boot failure drops (`emergency.target`) using `nofail`; cloud disk decommissioning order (`umount` -> fstab removal -> `mount -a` -> `lvremove` -> `vgremove` -> `pvremove`).
11. **Filesystem Layering (OverlayFS):** Architecture (`lowerdir`, `upperdir`, `workdir`, `merged`); how container image layers function natively on disk.
---
---
---
#### **Phase 3: Service Management, Systemd & Complete Boot Chain**

12. **The Complete Linux Boot Chain:** Stages: `UEFI/BIOS` -> `GRUB2` (kernel arguments, `init=/bin/bash` recovery) -> `initramfs` (early storage drivers) -> Kernel execution -> `systemd (PID 1)`.
13. **Systemd Service Unit Architecture:** Custom unit paths (`/etc/systemd/system/` vs `/lib/systemd/system/`); unit dependencies (`After=` vs `Wants=` vs `Requires=`); lifecycle controls (`Type=simple`, `User=`, `Restart=always`, `RestartSec=`, `LimitNOFILE=`).
14. **Process Sandboxing & Kernel Governance:** cgroups v2 integration directly inside unit files (`MemoryMax=`, `CPUQuota=`); filesystem isolation (`ProtectSystem=strict`, `ProtectHome=true`, `PrivateTmp=true`, `NoNewPrivileges=true`).
15. **Target Management & Boot Profiling:** Runlevels vs systemd targets (`multi-user.target`, `rescue.target`, `emergency.target`); boot latency diagnosis (`systemd-analyze`, `systemd-analyze blame`, `systemd-analyze critical-chain`).
16. **Binary Logging Engine (journald):** Structured logging via `journalctl` (`-u <unit>`, `-b`, `-p err..alert`, `--since`); vacuuming log storage allocations (`--vacuum-size=200M`).

---

#### **Phase 4: Low-Level Networking, Sockets & Packet Filtering**

17. **Modern Interface & Route Triage:** Replacing net-tools with iproute2 (`ip -br addr`, `ip route`, `ip neighbor`).
18. **Socket Lifecycles & State Machines:** Low-level socket inspection via `ss -tlpn` and `ss -s`; `TIME_WAIT` accumulation (ephemeral port exhaustion) vs `CLOSE_WAIT` leaks (unclosed application descriptors).
19. **Packet Filtering & Firewall Mechanics:** Netfilter packet lifecycle hooks (`PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`); `DROP` (silent timeout) vs `REJECT` (immediate RST); rule counter auditing (`iptables -L -n -v --line-numbers`) and rule insertion (`-I` vs `-A`).
20. **DNS Resolution Flow & systemd-resolved:** Application lookup chain (`/etc/nsswitch.conf` -> `/etc/hosts` -> `/etc/resolv.conf`); local loopback caching stub (`127.0.0.53:53`) and resolving upstream (Azure Wire Server: `168.63.129.16`); cache management via `resolvectl status` and `resolvectl flush-caches`.
21. **Kernel TCP Tuning via sysctl:** Tuning ephemeral ports (`net.ipv4.ip_local_port_range`), connection backlogs (`net.core.somaxconn`), and connection tracking limits.

---

#### **Phase 5: Core Enterprise Administration, Access Governance & Clocks**

22. **User Accounts & Group Lifecycles:** Internal formats (`/etc/passwd`, `/etc/shadow`, `/etc/group`); service accounts, system UIDs, group memberships, and least-privilege sudoers delegation (`visudo`).
23. **Advanced POSIX Access Control Lists (ACLs):** Overriding standard owner/group/other `rwxr-xr-x` masks; setting multi-user/multi-group fine-grained permissions via `getfacl` and `setfacl`.
24. **Scheduled Automation & Daemons:** Traditional cron (`/etc/cron.*`, `crontab`); modern `systemd.timer` units (monotonic timers, calendar events, missed execution catch-up, journal integration).
25. **Time Synchronization & Clock Drift:** Configuring `chronyd` (`chronyc tracking`, `chronyc sources`); preventing time drift failure modes across Kerberos tickets, OAuth/JWT tokens, TLS cert validation, and database consensus splits.
26. **Network Shared Storage (NFS):** Mounting NFS exports, tuning mount parameters (`hard` vs `soft`, `timeo`, `rsize`/`wsize`); recovering from dead NFS mounts (stale file handles) and safe boot integration (`_netdev` in `/etc/fstab`).

---

#### **Phase 6: Linux Namespaces & Building Containers from Scratch**

27. **The 6 Core Linux Isolation Namespaces:** `pid` (process tree virtualization), `net` (independent routing, firewall, veth), `mnt` (mount trees), `ipc` (message queues), `uts` (hostnames), and `user` (host-to-namespace root mapping).
28. **Hands-On Container Assembly:** Creating a container by hand without Docker or containerd; combining `unshare` + root filesystem tarball + `chroot` + `cgroups v2` resource quotas + `veth` pair attached to a bridge.

---

#### **Phase 7: OS Security, Hardening, Kernel Interfaces & Cloud Realities**

29. **Mandatory Access Control (MAC):** AppArmor (Ubuntu/Debian) and SELinux (RHEL/CentOS) architectures; enforcing vs complain/permissive modes; reading audit denial logs.
30. **Linux Capabilities Subsystem:** Breaking the root monolith into discrete privilege units (`CAP_NET_BIND_SERVICE`, `CAP_SYS_ADMIN`, `CAP_SETUID`); assigning capabilities directly to binaries via `setcap` and `getcap`.
31. **Host Authentication & SSH Hardening:** Production `sshd_config` standards (disabling password auth, disabling root login, key algorithms, `MaxAuthTries`); Pluggable Authentication Modules (PAM) structure in `/etc/pam.d/`.
32. **System Auditing & Core Dumps:** Kernel event auditing using `auditd` (`auditctl` rules, `ausearch`, `aureport`); managing core dumps (`coredumpctl`) and configuring process execution limits via `/etc/security/limits.conf` (`ulimit`).
33. **Cloud Infrastructure Internals (Azure Linux):** `cloud-init` boot stage execution, metadata consumption, and debugging `/var/log/cloud-init.log`; Azure Linux Agent (`waagent`) provisioning and swap creation; Instance Metadata Service (`IMDS`: `http://169.254.169.254/metadata/instance`).

---

#### **Phase 8: Advanced Performance Observability & Tracing (Brendan Gregg)**

34. **The USE Diagnostic Framework:** Systematic measurement across CPU, Memory, Storage, and Network: Utilization (% busy), Saturation (queue depth / delay), Errors (hardware/controller counters).
35. **Hardware Performance Counters (perf):** CPU Cycles, Instructions Retired, and Instructions Per Cycle (IPC); cache misses and memory stall profiling via `perf stat` and `perf top`.
36. **CPU Sampling & FlameGraph Visualization:** Recording on-CPU stack traces via `perf record`; generating and interpreting interactive SVG FlameGraphs to isolate inefficient application code.
37. **Off-CPU vs On-CPU Analysis:** Separating active CPU cycle burning from threads blocked waiting on locks, disk I/O, or thread synchronization.
38. **Dynamic Kernel Tracing (eBPF & bpftrace):** Writing `bpftrace` one-liners to trace disk I/O latency histograms (`biolatency`), live socket round-trip time (`tcprtt`), and execution tracing (`execsnoop`).
39. **Dynamic Syscall Tracing (strace / ltrace):** Attaching to running daemons (`strace -p <PID> -f`) to debug silent stalls, missing config files, or hung syscalls without source code.

---

#### **Phase 9: Universal Enterprise Networking Prerequisites**

40. **Layer 2 & Layer 3 Foundations:** Ethernet frame layout, MAC addressing, ARP protocol flow, and cache inspection; VLAN tagging (`802.1Q`); ICMP error messages (Type 3 Destination Unreachable, Type 11 Time Exceeded); MTU vs MSS clamping in encapsulated tunnels (`VXLAN`, `IPsec`, `WireGuard`).
41. **IP Addressing & Subnet Mathematics:** Binary IPv4, subnet masks, and prefix lengths (`/24`, `/27`, `/30`); manual network/broadcast calculation; RFC 1918 private scopes and overlap resolution; IPv6 fundamentals (ULA vs GUA, `/64` subnets, link-local addresses).
42. **Routing & Path Determination:** Longest Prefix Match (LPM) logic; default routing (`0.0.0.0/0`) and next-hop resolution; asymmetric routing drops in stateful systems; BGP fundamentals (ASN, Route Advertisements, `AS-Path`, `MED`, `Local Preference`).
43. **Network Address Translation (NAT/PAT):** SNAT (outbound internet egress masking) vs DNAT (inbound port forwarding); Port Address Translation (PAT) and SNAT port exhaustion mechanics.
44. **TCP State Machine & Transmission Controls:** 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) and 4-way termination; sliding window flow control, slow-start, packet retransmissions, keepalive timers, and TCP Reset (`RST`) injection.
45. **DNS Architecture & Query Resolution:** Recursive resolvers vs Authoritative nameservers; DNS record types (`A`, `AAAA`, `CNAME`, `PTR`, `TXT`, `SRV`, `NS`, `SOA`); TTL cache semantics, split-horizon DNS, and conditional forwarding rules.
46. **Transport Layer Security (TLS/PKI):** Public Key Infrastructure, Certificate Authorities (CA), and intermediate trust chains; TLS 1.3 handshake, Server Name Indication (SNI), and cipher suites; mutual TLS (mTLS); TLS Termination vs TLS Passthrough.
47. **Application Layer Protocols (HTTP Ecosystem):** HTTP/1.1 (head-of-line blocking) vs HTTP/2 (multiplexed binary framing) vs HTTP/3 (QUIC over UDP); critical proxy headers (`Host`, `X-Forwarded-For`, `X-Forwarded-Proto`).
48. **Load Balancing & Reverse Proxies:** Layer 4 transport hashing vs Layer 7 URL/header routing; active health probes and node draining; session persistence vs round-robin/least-connections; Forward proxies vs Reverse proxies.
49. **Stateful Packet Inspection & Application Firewalls:** 5-tuple tracking (Source IP/Port, Dest IP/Port, Protocol); connection tracking states (`NEW`, `ESTABLISHED`, `RELATED`, `INVALID`); Web Application Firewalls (WAF) Layer 7 payload inspection.
50. **Virtual Private Networks (VPNs):** IPsec architecture (IKEv1/IKEv2, SAs, ESP encapsulation); policy-based VPNs vs route-based VPNs (VTI with dynamic BGP); WireGuard cryptokey routing.
51. **Low-Level Network Packet Diagnostics:** Live packet analysis using `tcpdump`; reading PCAP traces in Wireshark (TCP stream reassembly, retransmission analysis); path and latency analysis using `mtr` and `traceroute`.

---

#### **Phase 10: Azure Cloud Enterprise Networking Architecture**

52. **Core Virtual Networks & IP Schemas:** Virtual Networks (VNets), Subnets, and reserved Azure IP offsets (first 4 IPs + last IP reserved); enterprise multi-region IP schema planning to prevent address collisions.
53. **Software-Defined Routing (UDRs):** Azure System Default Routes vs User Defined Routes (UDRs); Longest Prefix Match enforcement; next-hop types (`VirtualAppliance`, `VirtualNetworkGateway`, `Internet`, `None`).
54. **Ingress & Egress Traffic Filtering:** Network Security Groups (NSGs stateful 5-tuple rules, priority ordering); Application Security Groups (ASGs); Azure Firewall & NVAs (`ip_forward`, application FQDN filtering).
55. **Outbound Egress Scaling:** Azure NAT Gateway (resolving SNAT port exhaustion for microservices and AKS nodes); outbound rules on Azure Standard Load Balancer.
56. **Ingress Traffic Architecture:** Azure Load Balancer (Layer 4 regional traffic distribution); Azure Application Gateway (Regional Layer 7 reverse proxy, SSL offloading, URL routing, WAF); Azure Front Door (Global Layer 7 anycast reverse proxy with edge CDN/WAF); Azure Traffic Manager (DNS routing).
57. **Private Connectivity & Hybrid DNS:** Azure Private Endpoints (mapping PaaS to private IPs via VNIs); Azure Private DNS Zones & Virtual Network Links; Azure Private DNS Resolver (inbound/outbound endpoints with Forwarding Rulesets eliminating loops).
58. **Hybrid Cloud Interconnects:** Azure VPN Gateway (route-based active-active IPsec with BGP); Azure ExpressRoute (Private Peering vs Microsoft Peering, Global Reach, BGP communities).
59. **Enterprise Topology & Transit Routing:** Hub-and-Spoke topology (overcoming peering non-transitivity via central firewall); Azure Virtual WAN (vWAN fully meshed routing with native routing intents).
60. **Kubernetes (AKS) Network Mechanics:** Kubenet (pod IP masquerading) vs Azure CNI (direct VNet IP allocation); Azure CNI Overlay (efficient decoupling of pod CIDRs from VNet CIDRs); Cilium & eBPF integration bypassing iptables kube-proxy.

---

#### **Phase 11: The Principal-Grade Standard & Incident Triage**

61. **Architectural Design Standards:** Zero-overlap IP schematics across cloud, on-prem, and branch offices; deterministic transit routing ensuring traffic symmetry for stateful firewalls; resilient hybrid DNS topologies; fault domain isolation across availability zones.
62. **Universal Production Incident Triage Sequence:**

```bash
# STEP 1: DNS LAYER (Layer 7 Resolution)
# Did the hostname resolve? To which IP (Public vs Private Endpoint)?
dig @127.0.0.53 <host>
resolvectl query <host>

# STEP 2: ROUTING LAYER (Layer 3 Path Determination)
# Does the source subnet know how to reach the destination?
# Azure: Check "Effective Routes" on the NIC blade.
ip route show

# STEP 3: FIREWALL & FILTERING (Layer 4 Access)
# NSG: Check Inbound on destination AND Outbound on source.
# Host: Audit iptables / nftables byte counters for drops.
iptables -L -n -v --line-numbers

# STEP 4: SOCKET & APPLICATION (Layer 7 Host State)
# Is the process listening on 0.0.0.0 or restricted to 127.0.0.1?
ss -tlpn
# Has the process hit descriptor limits?
ulimit -n
