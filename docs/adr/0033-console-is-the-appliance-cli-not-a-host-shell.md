# VGA and serial provide the Appliance console, not a Host shell

VGA and serial never offer a Host shell. First boot the Appliance console provides the unauthenticated Bootstrap console (NIC list, ephemeral addressing, how to reach the UI). After Bootstrap, v1 uses the UI for routine configuration and retains the limited authenticated recovery menu in ADR-0058; the full Appliance CLI is deferred to v2 (ADR-0055).

Considered: getty + login + bash, then `fwos`. Rejected — that makes a Host shell the product, which v1 is not shipping (ADR-0032). If SSH is reopened, it is the same Appliance CLI, not a shell.

First-boot and post-bootstrap console remain one Host program, with mode selected by durable Bootstrap completion. This is not an addon or two daemons, and the limited v1 console does not imply a full configuration CLI. After GRUB the operator sees the Appliance console rather than kernel or systemd status (ADR-0035).
