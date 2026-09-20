# Join the forwarding netns only for Traffic NIC protocols

A process joins `fwd` iff it must send or receive on a Traffic NIC: `netd`, FRR, WireGuard, Kea, Unbound. UI, sshd, and addons stay out unless the manifest opts them in. CPU cost is not the test — DHCP is not 10G, but a DHCPDISCOVER arrives on the LAN Traffic NIC. A second netns for those protocols is an extra hop, not isolation.

Considered: putting “slow” daemons (Kea, Unbound) in another netns. Rejected; they are wire protocols on a Traffic NIC.

ADR-0060 adds a narrow exception for the temporary Host update worker: it uses `fwd` networking for WAN registry access without a delegated prefix, while the update controller stays in the Host netns. This does not make the worker a network-policy writer or move the UI into `fwd`; v1 has no network SSH (ADR-0032).
