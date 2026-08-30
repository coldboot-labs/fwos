# Console is the Appliance CLI, not a Host shell

VGA and serial never offer a Host shell. First boot they run the unauthenticated Bootstrap console (NIC list, ephemeral addressing, how to reach the UI). After Bootstrap they run the Appliance CLI: the operator authenticates as admin into that CLI. The operator alternative is the UI over HTTPS.

Considered: getty + login + bash, then `fwos`. Rejected — that makes a Host shell the product, which v1 is not shipping (ADR-0032). If SSH is reopened, it is the same Appliance CLI, not a shell.

First-boot and post-bootstrap console are **one Host program**, two modes, switched by Bootstrap complete on `/var`. Not two daemons. Not an addon.
