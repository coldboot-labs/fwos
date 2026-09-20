# Web UI is the primary v1 configuration interface; full CLI is v2

The UI is the primary place for post-bootstrap configuration in v1, with graphical controls for supported features backed by `netd`; the full Appliance CLI is deferred to v2. The Bootstrap console remains in v1, while the existing authenticated recovery requirement in ADR-0048 is separate from a full configuration CLI. Host update controls move into the v1 UI through the Host update program, and Desired state import/export remains available for backup and transfer, including optional passphrase encryption.

This replaces the restrictions that made status or file import/export the only post-bootstrap UI workflows, so adding features can extend the primary operator interface. The thin Bootstrap wizard, `netd`'s ownership of validation and application, optional Apply confirmation disabled by default, and the absence of network SSH remain.

Supersedes the UI and full-CLI scope clauses in ADR-0014, ADR-0041, and ADR-0045. Amends the operator-client and console assumptions in ADR-0022, ADR-0026, ADR-0028, ADR-0033, ADR-0034, ADR-0035, and ADR-0047.

ADR-0058 defines the limited authenticated recovery menu that remains in the v1 Appliance console.

ADR-0059 defines Review and apply as the normal configuration workflow, with a standalone Save and apply shortcut using the same `netd` application engine.
