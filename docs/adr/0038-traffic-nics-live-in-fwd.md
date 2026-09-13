# All Traffic NICs live in fwd

Every Traffic NIC, including a Management NIC, is moved into `fwd` when `netd` starts. The Management netns never owns a physical NIC. The UI stays in `mgmt` and is reached by nft DNAT of HTTPS to a veth between `fwd` and `mgmt`, from the addresses of the UI exposure set. Forwarded LAN↔WAN traffic does not use that path.

This is one story: the old stick exception is the only path. A dedicated Management NIC is an isolated role in `fwd`, not a netns class.

Supersedes the NIC-placement clauses of ADR-0006 (Management NIC moves to `mgmt`; first boot leaves NICs in the Host netns until Bootstrap). The UI still never joins `fwd`. Amends ADR-0003: `mgmt` owns the UI, not a Management NIC. Amends ADR-0004: the oneshot still only creates empty `fwd` and `mgmt`; `netd` moves NICs.

Considered: keep a Management NIC in `mgmt` for OOB isolation if `fwd` is compromised. Rejected for v1 — it was a second story, and the two-NIC WAN+LAN box cannot spare a port that leaves `fwd`.
