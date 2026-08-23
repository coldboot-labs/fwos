# Join the forwarding netns only for Traffic NIC protocols

A process joins `fwd` iff it must send or receive on a Traffic NIC: `netd`, FRR, WireGuard, Kea, Unbound. UI, sshd, and addons stay out unless the manifest opts them in. CPU cost is not the test — DHCP is not 10G, but a DHCPDISCOVER arrives on the LAN Traffic NIC. A second netns for those protocols is an extra hop, not isolation.

Considered: putting “slow” daemons (Kea, Unbound) in another netns. Rejected; they are wire protocols on a Traffic NIC.
