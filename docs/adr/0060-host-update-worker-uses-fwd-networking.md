# Host update worker uses Forwarding netns networking

The Host update program's controller and unix-socket API remain in the Host netns, while its temporary `bootc` worker uses the Forwarding netns network stack to reach the registry through the WAN. The worker retains access to the Host filesystem and deployment state required for the update; joining `fwd` is a network-placement choice, not a transfer of update ownership to `netd`. This satisfies the IPv6-only, no-prefix-delegation requirement without adding general LAN-to-WAN NAT66 or a separate outbound proxy service.

The worker does not configure interfaces, routes, or firewall policy: ongoing network administration remains with `netd`. The completed-Bootstrap gate, registry-pull-only v1 scope, and separate explicit reboot remain unchanged (ADRs 0019, 0026, and 0027).

Amends the update-networking assumptions in ADRs 0002, 0003, 0019, and 0022, implementing the connectivity requirement in ADR-0054. Verify the actual `bootc` execution path, child-process networking, DNS reachability from `fwd`, and Host filesystem/deployment access in the target VM before treating this mechanism as proven.
