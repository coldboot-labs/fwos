# netd is a built-in addon; the Host image ships a Network startup program

The Host image keeps host packages minimal: kernel, bootc, systemd, rescue `ip`/`nft`/`ethtool`, and the Network startup program. `netd` is a built-in addon joined to `fwd`, not a host package. `netd` is control plane (nft/netlink), not the 10G path, so throughput is not a reason to move it onto `/usr`. The Network startup program prepares the namespaces and fixed virtual links and moves Traffic NICs into `fwd` before `netd` starts (ADR-0052).

ADR-0052 supersedes the original restriction that the startup Host program only creates empty namespaces.
