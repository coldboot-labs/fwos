# Apply confirmation is optional and disabled by default

Apply confirmation is a configuration option, disabled by default: a successfully applied and durably saved Desired state is accepted without a further administrator acknowledgement. When enabled for a post-bootstrap UI apply, it requires acknowledgement through the UI within two minutes; otherwise `netd` restores the previous Accepted Desired state. Failure and interruption recovery remain mandatory regardless of this option; this optional safeguard addresses configuration that applies successfully but disconnects the administrator.

Complements ADR-0043 and ADR-0045. This is separate from Host update health and the exclusion of a Host update acknowledgement watchdog in ADR-0020.

Command-line acknowledgement is deferred with the full Appliance CLI to v2 (ADR-0055).

ADR-0062 allows any currently authenticated administrator to review and confirm the pending revision, records the applying and confirming administrators, and prevents another apply until confirmation or recovery finishes.
