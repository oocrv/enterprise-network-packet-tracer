# Network Configuration Notes

This document summarizes the addressing plan, intended configuration areas, and commands useful for checking the Cisco Packet Tracer lab. Adapt interface numbers and access-port ranges to match the actual cabling in your `.pkt` file.

## 1. Addressing Plan

| VLAN | Purpose | Subnet | Gateway |
|---:|---|---|---|
| 10 | HR | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Finance | `192.168.20.0/24` | `192.168.20.1` |
| 30 | IT | `192.168.30.0/24` | `192.168.30.1` |
| 40 | Operations | `192.168.40.0/24` | `192.168.40.1` |
| 50 | Servers | `192.168.50.0/24` | `192.168.50.1` |

Server settings:
- IPv4 address: `192.168.50.10`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.50.1`
- DNS server: `192.168.50.10`
- DNS record: `intranet.company.local → 192.168.50.10`

## 2. VLANs on the Switches

Create the required VLANs on both switches (if they are not already present):

```text
enable
configure terminal
vlan 10
 name HR
vlan 20
 name Finance
vlan 30
 name IT
vlan 40
 name Operations
vlan 50
 name Servers
end
copy running-config startup-config
```

Check VLANs and access-port assignments:

```text
show vlan brief
```

## 3. Trunk Verification

Check which switch interfaces are trunking:

```text
show interfaces trunk
```

On a 2960 switch, trunk configuration commonly includes `switchport mode trunk`. Configure the actual interfaces connected to the other switch or router, and make sure the required VLANs are allowed. Do not paste interface numbers without checking your topology.

## 4. Router-on-a-Stick Reference

The router uses subinterfaces on `GigabitEthernet0/0`. Confirm the physical interface connected to the switch before applying configuration.

```text
enable
configure terminal
interface gigabitEthernet 0/0
 no shutdown
!
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface gigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
!
interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
end
copy running-config startup-config
```

Useful checks:

```text
show ip interface brief
show running-config
```

Expected router subinterfaces should be operational when the physical link and switch trunk are configured correctly.

## 5. DHCP

Create a DHCP pool per client VLAN and exclude reserved addresses first. Example for VLAN 10 (repeat with the correct network, gateway, and pool name for the other VLANs):

```text
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp pool HR
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.50.10
```

Check leases and pool statistics:

```text
show ip dhcp binding
show ip dhcp pool
```

## 6. DNS and HTTP Server

In the Server-PT settings:
1. Set static IPv4 address `192.168.50.10/24`.
2. Set default gateway `192.168.50.1`.
3. Set DNS server to `192.168.50.10`.
4. Enable DNS and add an A record: `intranet.company.local` → `192.168.50.10`.
5. Enable HTTP.

From a client PC, test name resolution and connectivity:

```text
ping 192.168.50.10
ping intranet.company.local
```

Then open `http://intranet.company.local` in the PC's web browser.

## 7. Port Security (Document Only if Enabled)

If you configured port security on access ports, verify it with:

```text
show port-security
show port-security interface fastEthernet 0/2
show running-config
```

Example configuration for a known access-port range only (replace the range to match your topology):

```text
interface range fastEthernet 0/2-5
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

Do not apply this blindly to trunk/uplink ports. Verify the actual connected interfaces and check that the switch model in Packet Tracer supports the commands.

## 8. Evidence Screenshots

The repository's `screenshots/` folder contains the following evidence images. Keep these filenames consistent with the links in `README.md`:

1. `screenshots/01-topology.png` — entire topology with device names and links.
2. `screenshots/02-vlans.png` — `show vlan brief`.
3. `screenshots/03-trunks.png` — `show interfaces trunk`.
4. `screenshots/04-router.png` — router subinterfaces / `show ip interface brief`.
5. `screenshots/05-dhcp.png` — DHCP configuration or client addressing.
6. `screenshots/06-web-dns.png` — DNS record and HTTP service on the server.
7. `screenshots/07-ping.png` — connectivity and name-resolution tests.
8. `screenshots/08-port-security.png` — port-security verification, if configured.

Before presenting screenshots as proof of successful operation, verify that each screenshot shows the expected output and matches the current `.pkt` file.
