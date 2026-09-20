# v1 Bootstrap wizard is thin

The v1 wizard collects: admin, hostname, one WAN L2, one LAN L2, optional Management NIC, operator-chosen VLAN IDs, UI exposure (Management NIC locked on; LAN optional; total ≥1), static or DHCP, LAN prefix, DHCP pool, WAN v6/PD. Extra L2s, extra WANs, extra LAN VLANs, and unused NICs beyond that are not Bootstrap. `netd` still applies default policy when a WAN exists.

After Bootstrap the v1 UI is the primary configuration interface, with graphical controls for supported features and authenticated Desired state import/export (ADR-0055). The wizard's limited field list does not limit subsequent configuration through the UI. The full Appliance CLI is deferred to v2.

Amends the wizard field list in ADR-0014. ADR-0055 supersedes the original status-only and v1 full-CLI clauses; the thin Bootstrap wizard stands.
