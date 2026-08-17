# Troubleshooting 003 — Windows Firewall Blocked TEST01 ICMP

## Context

During Chapter 1 of my BelkaCorp homelab, I configured the temporary Ubuntu validation VM `TEST01` on the same Hyper-V Internal management network as my Windows host.

```text
Windows host  10.10.30.10/24
TEST01        10.10.30.20/24
Switch        vSW-MGMT (Internal)
```

Because both systems are in the same `/24` subnet, they should be able to communicate directly through the Hyper-V virtual switch without a router.

## Symptom

After configuring `TEST01` with a static IPv4 address, connectivity was asymmetric:

```text
Windows -> TEST01   success
TEST01  -> Windows  failed
```

The Ubuntu guest had the expected connected route:

```text
10.10.30.0/24 dev eth0 proto kernel scope link src 10.10.30.20
```

This showed that the guest address and local subnet route were present.

## Investigation

I inspected the Windows host-side management interface and firewall state.

The `vEthernet (vSW-MGMT)` interface was classified as:

```text
Name             Unidentified network
NetworkCategory  Public
IPv4Connectivity LocalNetwork
```

Windows Firewall was enabled for Domain, Private, and Public profiles.

Because Windows could initiate a ping to TEST01 but TEST01 could not initiate an ICMP echo request to Windows, I suspected the Windows host firewall rather than the Hyper-V switch or IP configuration.

## Resolution

I kept Windows Firewall enabled and created a narrowly scoped inbound ICMPv4 rule instead of disabling firewall protection globally.

```powershell
New-NetFirewallRule `
    -DisplayName "BelkaCorp MGMT - ICMPv4 Echo" `
    -Description "Allow ICMPv4 echo requests from the isolated BelkaCorp MGMT subnet to the Windows Hyper-V host." `
    -Direction Inbound `
    -Action Allow `
    -Protocol ICMPv4 `
    -IcmpType 8 `
    -InterfaceAlias "vEthernet (vSW-MGMT)" `
    -LocalAddress "10.10.30.10" `
    -RemoteAddress "10.10.30.0/24"
```

The rule is restricted to:

- inbound ICMPv4 echo requests;
- the Hyper-V MGMT adapter;
- the local management IP `10.10.30.10`;
- source addresses inside `10.10.30.0/24`.

## Final Verification

After creating the rule, I verified successful communication in both directions:

```text
Windows 10.10.30.10 <-> TEST01 10.10.30.20
                 ping success both ways
```

This confirmed that the Hyper-V Internal switch, host static address, guest static address, and same-subnet communication were functioning correctly. The failed reverse ping had been caused by the Windows host firewall policy.

## Evidence

- `screenshots/chapter-01/01-06c-test-vm-static-network-one-way-ping.png`
- `screenshots/chapter-01/01-06d-test-vm-bidirectional-ping.png`

The first screenshot records the asymmetric connectivity before remediation. The second records the scoped firewall rule and successful bidirectional ping after remediation.

## Lesson Learned

When connectivity works in one direction but not the other, I should not immediately assume the virtual switch or IP addressing is broken.

My troubleshooting sequence is:

```text
VERIFY IP + ROUTE
      |
      v
VERIFY DIRECTION OF FAILURE
      |
      v
INSPECT HOST/GUEST FIREWALL
      |
      v
MAKE NARROW CHANGE
      |
      v
RETEST BOTH DIRECTIONS
```

This incident also reinforced why I should prefer a specific firewall exception over disabling a firewall during troubleshooting.
