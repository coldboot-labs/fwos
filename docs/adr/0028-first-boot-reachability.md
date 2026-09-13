# First-boot reachability is the UI and Bootstrap console, not SSH

Bootstrap is complete when the admin exists and interface roles plus UI exposure have been applied. v1 has no network SSH at any phase (ADR-0032). First-boot UI bind policy is non-global: IPv4 RFC1918, IPv4 link-local `169.254/16`, IPv6 ULA, IPv6 link-local — not IPv4 global unicast, not IPv6 GUA, not CGNAT `100.64/10`. How the UI becomes reachable is ADR-0039 (console-opted overlay on one Traffic NIC in `fwd`, not a Host-netns listen on every NIC).

After bootstrap, VGA and serial run the Appliance CLI, not a Host shell (ADR-0033). QEMU must prove the published path: serial Bootstrap console, HTTPS to the UI, bootstrap completes; the UI is in `mgmt` and there is no network SSH.

Steady state: the UI in `mgmt`, never `fwd` (ADR-0006, ADR-0038).

Revised by ADR-0039.
