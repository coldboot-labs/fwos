# UI exposure is non-WAN interface addresses

The operator-facing UI address is the interface address of each member of the UI exposure set. An L2 is a parent plus tag (untagged or a VLAN). WAN and LAN must not share one. v1 never exposes the UI on a WAN. A Management NIC owns its whole parent, is isolated (no LAN services, no forward), is addressed on-link with a static prefix and no gateway, and is always in the set. LAN membership is chosen by the operator. The set must be non-empty. v1 Desired state requires at least one WAN and at least one LAN; a Management NIC is optional.

v1 has no network SSH (ADR-0032); when SSH is reopened it follows this set, not a second path.

Amends ADR-0011: `netd` owns UI exposure, not a "mgmt exception" special case.
