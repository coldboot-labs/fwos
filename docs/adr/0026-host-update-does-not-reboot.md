# Host update stages; reboot is explicit

The Host update program pulls and stages the next bootc deployment and does not reboot. It reports that a reboot is required. The v1 UI offers reboot as a separate explicit operator action (ADR-0055); the full Appliance CLI is deferred to v2. Forwarding keeps running until that reboot. Failed-boot rollback (ADR-0020) applies to that reboot, not to staging. No scheduled window in v1.
