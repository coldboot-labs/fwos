# Interrupted configuration application recovers or blocks forwarding

On a bootstrapped appliance, the previous Accepted Desired state is retained durably until its replacement is accepted. An interrupted apply, including power loss, restores that previous revision; if restoration also fails, FWOS blocks forwarded traffic and provides authenticated console recovery. Recovery must not open the firewall or return an already-bootstrapped appliance to unauthenticated Bootstrap.

Extends ADR-0043. Optional Apply confirmation in ADR-0047 does not disable this recovery behavior.

ADR-0058 additionally retains the previous accepted revision after its replacement is accepted, for manual restoration through the authenticated Appliance console.

ADR-0062 serializes applies through recovery. Restoring network Desired state preserves current Identity configuration under ADR-0063.
