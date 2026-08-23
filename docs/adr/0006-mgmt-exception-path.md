# In-band SSH/UI never join fwd

SSH and the UI run in `mgmt`. On a box with a Management NIC, that NIC moves to `mgmt`. On a one-NIC stick, `netd` programs nft DNAT of the designated mgmt address/ports to a veth or macvlan between `fwd` and `mgmt`. Forwarded LAN↔WAN traffic does not use that path.

First boot creates empty `fwd` and `mgmt` and does not move NICs, so SSH can stay in the Host netns until a Management NIC or the exception exists; then sshd runs in `mgmt`.

Considered: sshd in `fwd` on the stick only; leaving the parent NIC out of `fwd` and stacking VLANs across netns. Rejected — two mgmt stories, or a netns hop on the 10G path.
