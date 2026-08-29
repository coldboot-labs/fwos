# Host update stages; reboot is explicit

The Host update program pulls and stages the next bootc deployment and does not reboot. It reports that a reboot is required. Appliance CLI (and later the UI) reboots only when the operator asks. Forwarding keeps running until that reboot. Failed-boot rollback (ADR-0020) applies to that reboot, not to staging. No scheduled window in v1.
