# Three persistent network namespaces

The appliance has three persistent netns: the Host netns (emptied of Traffic NICs and Management NICs), `mgmt` (Management NIC, SSH, UI), and `fwd` (Traffic NICs). Addon manifest `mgmt` joins the Management netns; `fwd` joins the Forwarding netns; `none` is a private empty netns.

Considered: two netns with `mgmt` meaning the Host netns. Rejected — SSH/UI and the Management NIC live in a dedicated `mgmt` netns, not with PID 1.

The Host netns has a veth to `mgmt` and a default route via `mgmt` so `bootc` and image pulls work. `mgmt` is secondary and may be slow. Physical Traffic NICs and Management NICs still do not live in the Host netns. An air-gapped Host netns (`NetworkNamespacePath=mgmt` on every networked unit) was considered and not taken.
