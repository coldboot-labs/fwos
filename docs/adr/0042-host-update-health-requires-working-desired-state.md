# Host update health requires restoring previously working Desired state

After a Host update, a new bootc deployment is accepted only when the default target is reached, `fwd` and `mgmt` exist, `netd` is running, previously working Desired state has been restored, and the local services required by that Desired state have started. Failure triggers automatic rollback to the previous bootc deployment: process and namespace checks alone cannot establish that the new Release works. An unplugged WAN, an unavailable ISP, or other upstream unreachability does not trigger rollback.

Supersedes ADR-0023. The automatic rollback requirement in ADR-0020 stands.

ADR-0046 defines the configuration and persistent daemon state compatibility required for the previous Release to work after rollback.
