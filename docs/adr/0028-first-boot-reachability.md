# First-boot reachability is the UI and Bootstrap console, not SSH

Bootstrap is complete when the admin exists and NIC placement has been applied (Management NIC or stick exception). Until then, a published Disk image or Installer has no network SSH. The UI listens in the Host netns only on non-global addresses: IPv4 RFC1918, IPv4 link-local `169.254/16`, IPv6 ULA, IPv6 link-local — not IPv4 global unicast, not IPv6 GUA, not CGNAT `100.64/10`. It always listens on link-local on every Host-netns NIC.

An unauthenticated Bootstrap console Host program on tty1 and serial lists those NICs and sets ephemeral Host-netns addressing (static, DHCP, or SLAAC) so a human can reach the UI without typing a scoped IPv6 URL. That addressing is not Desired state and is discarded when NICs move. The Bootstrap console is not Appliance CLI, not `netd`, and is gone after bootstrap. Dev/test Disk images may still inject an SSH key; that is not the published artifact. QEMU must still prove the published path: serial Bootstrap console, HTTPS to the UI, bootstrap completes, SSH works in `mgmt` and not in the Host netns.

Steady state is unchanged: SSH and the UI in `mgmt`, never `fwd` (ADR-0006).
