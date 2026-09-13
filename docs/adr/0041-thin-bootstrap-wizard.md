# v1 Bootstrap wizard is thin

The v1 wizard collects: admin, hostname, one WAN L2, one LAN L2, optional Management NIC, operator-chosen VLAN IDs, UI exposure (Management NIC locked on; LAN optional; total ≥1), static or DHCP, LAN prefix, DHCP pool, WAN v6/PD. Extra L2s, extra WANs, extra LAN VLANs, and unused NICs beyond that are not Bootstrap. `netd` still applies default policy when a WAN exists.

After Bootstrap the v1 UI is status only (ADR-0014). The Appliance CLI still applies full Desired state, including L2s the wizard did not collect. A richer CLI UX is later and is not Bootstrap. A later admin UI that edits Desired state reopens ADR-0014; it is not a v1 hole.

Amends the wizard field list in ADR-0014. The status-only and full-CLI clauses stand.
