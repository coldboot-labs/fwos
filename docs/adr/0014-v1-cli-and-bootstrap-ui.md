# v1 uses the UI for configuration; the full Appliance CLI is v2

v1's UI is the primary post-bootstrap configuration interface, with graphical controls for supported features backed by `netd`. The UI is a built-in addon in `mgmt` (ADR-0039) and a v1 client of the Host update program (ADR-0022). The full Appliance CLI is deferred to v2 (ADR-0055).

The v1 UI Bootstrap wizard remains thin (ADR-0041). After Bootstrap, graphical configuration, status, and authenticated Desired state import/export are available through the UI; `netd` remains responsible for validation and application. Import/export remains useful for backup and transfer alongside graphical configuration (ADR-0045).

The Appliance console remains a Host program for Bootstrap and authenticated recovery (ADR-0033, ADR-0048), without full configuration commands in v1. Hand-editing TOML remains a break-glass operation rather than the routine configuration workflow.

ADR-0055 supersedes the original full-CLI-first, status-only, and import/export-only UI scope. This change does not introduce network SSH or independently add Addon management to v1.
