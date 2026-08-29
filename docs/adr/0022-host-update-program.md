# Host update is a Host program, not netd

A Host program in the Host netns wraps `bootc` and listens on a unix socket on `/var`. Appliance CLI (v1) and the UI (later) are clients of that socket. `netd` stays the Desired-state API in `fwd` and does not touch ostree. Executing `bootc` from the CLI process, or treating `systemctl start` as the product API, would leave the later UI with no seam.
