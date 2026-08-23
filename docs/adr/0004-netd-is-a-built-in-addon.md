# netd is a built-in addon; the host ships a netns oneshot

The host image has as few *packages* as possible: kernel, bootc, systemd, rescue `ip`/`nft`/`ethtool`, and one Host program oneshot that creates empty `fwd` and `mgmt`. `netd` is a built-in addon joined to `fwd`, not a host package. `netd` is control plane (nft/netlink), not the 10G path, so throughput is not a reason to move it onto `/usr`.
