# v1 has an Appliance CLI and a bootstrap UI, both netd clients

v1 ships an Appliance CLI and a basic UI, both in `mgmt`, both talking only to `netd`. Neither is a Host program.

The UI is **bootstrap-only**: hostname and admin credential; classify NICs (Management NIC vs Traffic NICs, stick VLAN IDs); WAN v4/v6 addressing and PD; LAN prefix, DHCP pool, and v6 from PD or static; default policy (NAT44 if v4 WAN, WAN inbound deny except the mgmt exception, LAN outbound allow). No v1 UI for a firewall rule editor, WireGuard, qdiscs, extra static routes, or addons.

The CLI can apply **full Desired state** (including WG, nft beyond the default, static routes). Hand-edit TOML is break-glass.
