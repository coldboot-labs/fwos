# Host update is a Host program, not netd

A Host program's controller in the Host netns wraps `bootc` and listens on a unix socket on `/var`; its temporary worker uses `fwd` networking under ADR-0060. The UI is a v1 client of that socket; the full Appliance CLI is deferred to v2 (ADR-0055). `netd` stays the Desired-state API in `fwd` and does not touch ostree. The Host update program owns the operation so operator clients do not execute `bootc` themselves or treat `systemctl start` as the product API.

ADR-0054 requires Host updates to work over an IPv6-only WAN without prefix delegation while preserving this program's ownership of the update operation.
