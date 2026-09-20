# Review and apply and quick apply share one configuration engine

The normal v1 UI workflow stages related edits as Draft Desired state, then lets the administrator review and apply them together. A standalone change also offers a Save and apply shortcut, using the same revision-based validation and application engine in `netd` rather than a separate immediate-mutation path. Neither action may silently apply another administrator's unfinished changes.

Both workflows retain validation before mutation, mandatory failure and interruption recovery, and optional Apply confirmation disabled by default (ADRs 0043, 0047, and 0048). Saving a draft is not activation or acceptance, and review before application is distinct from confirmation after application.

Extends ADR-0055's UI-first configuration scope and ADR-0011's ownership of Desired state. ADR-0061 defines private durable drafts, stale-revision checks, and the restriction on quick apply when the administrator has pending edits; ADR-0062 serializes application through confirmation and recovery.
