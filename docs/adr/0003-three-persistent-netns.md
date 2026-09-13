# Three persistent network namespaces

The appliance has three persistent netns: the Host netns (emptied of Traffic NICs at boot), `mgmt` (UI), and `fwd` (Traffic NICs, including a Management NIC). Addon manifest `mgmt` joins the Management netns; `fwd` joins the Forwarding netns; `none` is a private empty netns.

Considered: two netns with `mgmt` meaning the Host netns. Rejected — the UI lives in a dedicated `mgmt` netns, not with PID 1.

The Host netns has a veth to `mgmt` and a default route via `mgmt` so `bootc` and image pulls work. `mgmt` is secondary and may be slow. Physical NICs do not live in the Host netns or in `mgmt`. An air-gapped Host netns (`NetworkNamespacePath=mgmt` on every networked unit) was considered and not taken.

Amended by ADR-0032 (no v1 SSH) and ADR-0038 (Management NIC stays in `fwd`).
