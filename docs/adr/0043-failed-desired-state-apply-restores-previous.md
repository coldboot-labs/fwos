# Failed Desired state changes restore the previous configuration

After Bootstrap, when an operator changes Desired state, `netd` validates it before mutating live configuration. If application still fails, `netd` restores the previous Accepted Desired state so the operator is not left to repair a partially applied change. A brief traffic interruption during recovery is acceptable.

Amends ADR-0011. Host update failure remains governed by ADR-0042.

ADR-0047 adds optional Apply confirmation, disabled by default. ADR-0048 defines recovery from an interrupted apply or failed restoration on a bootstrapped appliance.

Incomplete Bootstrap attempts have no previous Accepted Desired state and instead follow ADR-0051.
