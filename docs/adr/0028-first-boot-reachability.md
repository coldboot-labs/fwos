# First-boot reachability is the UI and Bootstrap console, not SSH

Bootstrap is complete when the admin exists and NIC placement has been applied (Management NIC or stick exception). v1 has no network SSH at any phase (ADR-0032). The UI listens in the Host netns only on non-global addresses: IPv4 RFC1918, IPv4 link-local `169.254/16`, IPv6 ULA, IPv6 link-local — not IPv4 global unicast, not IPv6 GUA, not CGNAT `100.64/10`. It always listens on link-local on every Host-netns NIC.

An unauthenticated Bootstrap console on tty1 and serial lists those NICs and sets ephemeral Host-netns addressing (static, DHCP, or SLAAC) so a human can reach the UI without typing a scoped IPv6 URL. That addressing is not Desired state and is discarded when NICs move. After bootstrap, VGA and serial run the Appliance CLI, not a Host shell (ADR-0033). QEMU must prove the published path: serial Bootstrap console, HTTPS to the UI, bootstrap completes; the UI is in `mgmt` and there is no network SSH.

Steady state: the UI in `mgmt`, never `fwd` (ADR-0006).
