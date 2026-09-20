# v1 UI imports and exports Desired state

After Bootstrap, the UI provides status and authenticated import/export of full Desired state, with validation and an explicit apply step through `netd`. Import/export remains available for backup and transfer alongside the graphical configuration introduced by ADR-0055. That later decision also brings Host update into the v1 UI; Addon management is not independently added by these changes.

Supersedes the original status-only restriction in ADR-0014 and ADR-0041. ADR-0055 subsequently supersedes the import/export-only UI scope and defers the full Appliance CLI to v2; the thin Bootstrap wizard and no-network-SSH decision in ADR-0032 stand.

ADR-0047 defines optional Apply confirmation, disabled by default. ADR-0050 defines secret-bearing exports and optional encryption.
