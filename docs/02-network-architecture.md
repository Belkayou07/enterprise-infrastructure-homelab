# Chapter 2 — Network Architecture and IP Design

## Goal

I am defining how the BelkaCorp networks should behave before I deploy the firewall/router that will connect them. Chapter 1 established the Hyper-V platform and virtual switches; this chapter turns those isolated Layer-2 segments into a documented network design with clear addressing, gateway, routing, DHCP, and DNS ownership decisions.

## Why This Matters in a Company

A working virtual switch does not by itself create a usable enterprise network. I need to understand which systems belong to each subnet, which traffic is local, which traffic requires routing, which device owns each default gateway, and how address and name services will be provided. Defining those responsibilities before deployment reduces accidental routing, overlapping addresses, and unclear security boundaries.

## Starting State

At the start of Chapter 2, the Hyper-V platform is complete and the following virtual switches already exist:

| Switch | Type | Current role |
|---|---|---|
| Default Switch | Internal | Planned upstream/WAN side for `FW01` |
| `vSW-MGMT` | Internal | Management network shared by the Windows host and lab VMs |
| `vSW-SERVERS` | Private | Future server network |
| `vSW-USERS` | Private | Future user/client network |

The Windows host currently has `10.10.30.10/24` on `vEthernet (vSW-MGMT)`. `TEST01` remains available at `10.10.30.20/24` as a temporary validation VM.

`FW01` has not been deployed yet, so the planned gateway addresses `10.10.10.1`, `10.10.20.1`, and `10.10.30.1` do not yet exist.

## Chapter 2 Plan

- [x] 2.1A — Verify the Windows host network baseline
- [x] 2.1B — Verify the TEST01 guest network baseline
- [x] 2.2 — Confirm subnet and address allocation
- [ ] 2.3 — Define Layer-2 and Layer-3 boundaries
- [ ] 2.4 — Define gateway and routing behavior
- [ ] 2.5 — Define DHCP and DNS ownership
- [ ] 2.6 — Produce the final network implementation plan
- [ ] 2.7 — Complete the Chapter 2 audit

## 2.1A — Windows Host Network Baseline

I verified the host-side network state before introducing any routing device.

### Virtual switches

The host reports one instance of each planned switch:

```text
Default Switch  Internal
vSW-MGMT        Internal
vSW-SERVERS     Private
vSW-USERS       Private
```

### Management address

The Windows management interface is manually configured as:

```text
Interface  vEthernet (vSW-MGMT)
IPv4       10.10.30.10/24
Origin     Manual
```

### Management routes

Windows has an on-link route for the management subnet:

```text
10.10.30.0/24 -> next hop 0.0.0.0
```

The `0.0.0.0` next-hop value in this route means the destination is directly reachable on that interface. Windows does not need a router to reach another host in `10.10.30.0/24`; it can resolve the destination at Layer 2 and send directly through `vSW-MGMT`.

Windows also created the normal interface-local host, broadcast, multicast, and limited-broadcast routes associated with this IPv4 interface.

### Default route

The host's IPv4 default route remains:

```text
Interface  Wi-Fi
Next hop   192.168.0.1
```

There is no default route through `vEthernet (vSW-MGMT)`. This confirms that the isolated lab management network does not replace the Windows host's normal Internet path.

### Evidence

![Windows pre-router network baseline](../screenshots/chapter-02/02-01a-network-baseline-host.png)

This evidence captures the final switch inventory, the host's manual `10.10.30.10/24` management address, its directly connected MGMT route, and the normal Wi-Fi default route.

## 2.1B — TEST01 Guest Network Baseline

I then verified the same management network from inside the Linux guest.

### Guest address and connected route

TEST01 reports:

```text
eth0  UP  10.10.30.20/24
```

Its IPv4 routing table contains:

```text
10.10.30.0/24 dev eth0 proto kernel scope link src 10.10.30.20
```

This means TEST01 also treats `10.10.30.0/24` as directly connected.

### Route to the Windows host

For destination `10.10.30.10`, Linux selected:

```text
10.10.30.10 dev eth0 src 10.10.30.20
```

There is no `via` gateway in this decision because both systems are in the same `/24` subnet.

### Layer-2 neighbor resolution

After successful communication with the Windows host, TEST01 learned the host's Layer-2 neighbor mapping:

```text
10.10.30.10 lladdr 00:15:5d:38:01:02 REACHABLE
```

This confirms that TEST01 resolves the destination IP to the Windows host's MAC address and delivers the frame directly through `vSW-MGMT`.

### Connectivity

The guest successfully reached the Windows host:

```text
4 packets transmitted
4 received
0% packet loss
```

### Route to the future SERVERS subnet

When TEST01 tried to resolve a route toward the future server address `10.10.20.20`, Linux returned:

```text
RTNETLINK answers: Network is unreachable
```

This is expected in the pre-router state. TEST01 has no route for `10.10.20.0/24`, no default gateway, and no `FW01` interface yet to forward traffic between MGMT and SERVERS.

### Evidence

![TEST01 pre-router network baseline](../screenshots/chapter-02/02-01b-network-baseline-test01.png)

This evidence shows TEST01's `10.10.30.20/24` address, its directly connected MGMT route, direct route selection to the Windows host, successful 4/4 connectivity, the learned Layer-2 neighbor entry, and the expected unreachable result for the future SERVERS subnet.

## What the Pre-Router Baseline Proves

Both systems independently make the same decision for management traffic:

```text
Windows 10.10.30.10          TEST01 10.10.30.20
        |                            |
        +-------- vSW-MGMT ----------+
               10.10.30.0/24

Same subnet -> direct Layer-2 delivery
Different subnet -> no BelkaCorp route yet
```

A router is therefore unnecessary for communication inside `10.10.30.0/24`, but a Layer-3 device will be required when traffic must move between MGMT, SERVERS, and USERS.

## 2.2 — Subnet and Address Allocation

I use the private `10.10.0.0/16` block as the BelkaCorp address space and divide the current lab into three `/24` networks.

| Network | Subnet | Planned gateway | Main use |
|---|---|---|---|
| USERS | `10.10.10.0/24` | `10.10.10.1` | User/client systems |
| SERVERS | `10.10.20.0/24` | `10.10.20.1` | Infrastructure and application servers |
| MGMT | `10.10.30.0/24` | `10.10.30.1` | Administration and management access |

For each `/24`, the `.0` address is the network address and `.255` is the broadcast address, leaving `.1` through `.254` as usable host addresses.

The `.1` address is not reserved by IPv4 itself; I intentionally reserve it in my design as the default-gateway address on each subnet so the addressing convention remains predictable.

### Addressing convention

```text
.1           FW01 / default gateway
.2 - .9      reserved infrastructure
.10 - .49    static infrastructure and servers
.50 - .99    future static systems
.100 - .199  dynamic clients where DHCP is appropriate
.200 - .254  spare / future use
```

This is a design convention rather than a protocol requirement, and I only use the ranges where they make sense for the specific subnet.

### Current planned assignments

```text
USERS — 10.10.10.0/24
10.10.10.1         FW01
10.10.10.100-.199  planned DHCP pool

SERVERS — 10.10.20.0/24
10.10.20.1         FW01
10.10.20.10        DC01
10.10.20.20        LNX01
10.10.20.30        MON01

MGMT — 10.10.30.0/24
10.10.30.1         FW01
10.10.30.10        Windows host
10.10.30.20        TEST01 (temporary)
```

The third octet also reflects the network purpose (`10` USERS, `20` SERVERS, `30` MGMT), which makes addresses easier to recognize during configuration and troubleshooting.

## Status

**Chapter 2 is IN PROGRESS.** The pre-router baseline and the subnet/address allocation are now defined. The next step is to define the Layer-2 and Layer-3 boundaries of the BelkaCorp topology.
