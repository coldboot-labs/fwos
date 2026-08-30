# v1 has an Appliance CLI (Host program) and a management UI (addon)

v1 ships an Appliance CLI and a UI, both talking to `netd` for Desired state. The Appliance CLI is a Host program on VGA/serial (ADR-0033): first-boot mode is the Bootstrap console; after Bootstrap it is the admin CLI. It is also a client of the Host update program (ADR-0022). The UI is a built-in addon, not a Host program. The v1 UI is not a Host update client. The UI starts in the Host netns and continues in `mgmt` after bootstrap (ADR-0028, ADR-0030).

The v1 UI **Bootstrap wizard**: admin, hostname, classify NICs into `fwd` or `mgmt` (including one-NIC stick with VLANs), static or DHCP, LAN prefix, DHCP pool, WAN v6/PD. `netd` applies default policy when a WAN exists (no picker). After Bootstrap, v1 UI is **status only**. No v1 UI rule editor, WireGuard, qdiscs, extra static routes, addons, or Host update.

The CLI can apply **full Desired state** (including WG, nft beyond the default, static routes, and anything the wizard did not collect). Hand-edit TOML is break-glass.

Considered: bootstrap-only UI; post-bootstrap re-edit of the wizard fields. Rejected — after first boot the v1 UI is a status page; change of config is the Appliance CLI.
