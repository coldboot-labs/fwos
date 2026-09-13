# In-band UI never joins fwd

The UI runs in `mgmt` and never joins `fwd`. v1 has no network SSH (ADR-0032); sshd is not a v1 inhabitant of `mgmt`.

NIC placement and first-boot UI location are ADR-0038 and ADR-0039. The in-band path is no longer a stick exception: every UI packet is DNAT from a UI exposure address in `fwd` onto a veth into `mgmt`. Forwarded LAN↔WAN traffic does not use that path. A Management NIC stays in `fwd` (ADR-0038, ADR-0040).

Considered: sshd in `fwd` on the stick only; leaving the parent NIC out of `fwd` and stacking VLANs across netns. Rejected — two mgmt stories, or a netns hop on the 10G path.

Superseded in part by ADR-0038 and ADR-0039. The title stands: the UI never joins `fwd`.
