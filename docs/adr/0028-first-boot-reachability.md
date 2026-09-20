# First-boot reachability is the UI and Bootstrap console, not SSH

Bootstrap is complete when the admin exists and interface roles plus UI exposure have been applied. v1 has no network SSH at any phase (ADR-0032). First-boot UI bind policy permits IPv4 RFC1918 and IPv6 ULA only: not IPv4 global unicast, IPv6 GUA, CGNAT `100.64/10`, or either family's link-local addresses (ADR-0044, ADR-0049). How the UI becomes reachable is ADR-0039 (console-opted overlay on one Traffic NIC in `fwd`, not a Host-netns listen on every NIC).

After Bootstrap, the UI is the primary configuration interface and VGA/serial retain the Appliance console without a Host shell or full configuration CLI in v1 (ADR-0033, ADR-0055). QEMU must prove the published path: serial Bootstrap console, HTTPS to the UI, Bootstrap completes; the UI is in `mgmt` and there is no network SSH.

Steady state: the UI in `mgmt`, never `fwd` (ADR-0006, ADR-0038).

Revised by ADR-0039, ADR-0044, and ADR-0049.

ADR-0051 defines durable Bootstrap completion and clean retries of incomplete attempts.
