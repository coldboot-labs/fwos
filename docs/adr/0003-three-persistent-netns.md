# Three persistent network namespaces

The appliance has three persistent netns: the Host netns (emptied of Traffic NICs at boot), `mgmt` (UI), and `fwd` (Traffic NICs, including a Management NIC). Addon manifest `mgmt` joins the Management netns; `fwd` joins the Forwarding netns; `none` is a private empty netns.

Considered: two netns with `mgmt` meaning the Host netns. Rejected — the UI lives in a dedicated `mgmt` netns, not with PID 1.

The Host netns retains a veth to `mgmt` and a default route via `mgmt`. `mgmt` is secondary and may be slow. Physical NICs do not live in the Host netns or in `mgmt`. An air-gapped Host netns (`NetworkNamespacePath=mgmt` on every networked unit) was considered and not taken.

Amended by ADR-0032 (no v1 SSH) and ADR-0038 (Management NIC stays in `fwd`).

ADR-0054 requires Host update connectivity over an IPv6-only WAN without a delegated prefix. The Host-to-`mgmt` route alone does not satisfy that requirement when the Host netns has only internal ULA addressing; ADR-0060 therefore places the temporary `bootc` worker in `fwd` while keeping its controller in the Host netns.
