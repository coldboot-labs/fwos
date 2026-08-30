# In-band UI never joins fwd

The UI runs in `mgmt`. On a box with a Management NIC, that NIC moves to `mgmt`. On a one-NIC stick, `netd` programs nft DNAT of the designated mgmt address/ports (v1: the UI's HTTPS, not SSH) to a veth or macvlan between `fwd` and `mgmt`. Forwarded LAN↔WAN traffic does not use that path. v1 has no network SSH (ADR-0032); sshd is not a v1 inhabitant of `mgmt`.

First boot creates empty `fwd` and `mgmt` and does not move NICs. The UI is what listens in the Host netns until bootstrap (ADR-0028). After a Management NIC or the stick exception exists, the UI runs in `mgmt`.

Considered: sshd in `fwd` on the stick only; leaving the parent NIC out of `fwd` and stacking VLANs across netns. Rejected — two mgmt stories, or a netns hop on the 10G path.
