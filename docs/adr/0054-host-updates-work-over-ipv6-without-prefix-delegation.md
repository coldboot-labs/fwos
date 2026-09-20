# Host updates work over IPv6 without prefix delegation

If the container registry is reachable through the WAN's IPv6 connection, Host updates must work even when the WAN is IPv6-only and no prefix is delegated or otherwise routed to the internal namespaces. Provide a narrowly scoped outbound path for update traffic so the appliance can update itself without depending on extra upstream address space. General LAN-to-WAN NAT66 remains outside v1.

Amends the connectivity requirements in ADR-0003, ADR-0019, and ADR-0022. The translation policy in ADR-0009 and the completed-Bootstrap gate in ADR-0027 stand.

ADR-0060 selects a temporary update worker using `fwd` networking, while retaining the controller and update API in the Host netns.
