# Host update preserves rollback-compatible state

A Host update preserves a pre-update Accepted Desired state revision, and persistent daemon state must remain readable by the previous Release. v1 does not permit irreversible state migrations, because reverting the bootc deployment leaves shared `/var` intact and the previous binaries must still be able to operate. Rollback can therefore restore the preserved network configuration without depending on the failed Release to reverse its state changes.

Complements ADR-0015 and ADR-0042.

ADR-0063 excludes Identity configuration from network revision restoration: current accounts, credentials, and authentication settings remain in effect across rollback and must be readable by the previous Release.
