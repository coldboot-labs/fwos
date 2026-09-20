# Configuration applies are serialized through confirmation and recovery

Only one Desired state apply may be in flight, including any Apply confirmation or recovery window; other administrators may continue drafting but cannot apply until it finishes. Any currently authenticated administrator may review and confirm the pending revision, and FWOS records who applied it and who confirmed it. This keeps one unambiguous previous Accepted Desired state as the recovery target and prevents a timeout from undoing a second administrator's later apply.

Extends ADRs 0047, 0048, and 0059. Apply confirmation remains optional and disabled by default; serialization and actor attribution apply to both Review and apply and Save and apply.
