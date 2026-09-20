# UI always runs in mgmt; first-boot overlay is console-gated

The UI is the same built-in addon and always joins `mgmt`. There is no Host-netns UI phase. Traffic NICs are already in `fwd` (ADR-0038). Until the Bootstrap console opts exactly one Traffic NIC, nothing DNATs 443. Opt sets ephemeral addressing on that NIC's untagged L2 (static, DHCP, or SLAAC), persists outside Desired state until Bootstrap, and can be replaced. Apply discards it and installs UI exposure. First-boot bind policy is still non-global (ADR-0028). Un-opted NICs get no DHCP and no SLAAC/RA.

VGA/serial is the only door until opt. A one-NIC wizard warns that untagged first-boot HTTPS will vanish if untagged becomes WAN; it does not block apply.

Supersedes ADR-0030. Revises ADR-0028: Bootstrap complete is admin plus interface roles and UI exposure, not "Management NIC or stick"; the UI does not listen in the Host netns; there is no all-NIC link-local UI. Revises ADR-0029: no auto DHCP or SLAAC on un-opted NICs in `fwd`.

Considered: keep two-phase UI until WAN is known; auto link-local on every NIC. Rejected — two-phase was the Host-netns NIC story; spraying 443 on every port before WAN is classified is the thing console-gating exists to stop.

ADR-0051 amends failure handling: an incomplete Bootstrap attempt is discarded and the console-opted Bootstrap environment is restored before unauthenticated setup resumes.
