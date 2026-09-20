# Two bootc deployments, not A/B partitions

The Host image uses bootc's current and previous deployments on one root, sharing `/var`. There are no A/B root partitions, slot metadata, or a second updater (RAUC, sysupdate). Physical A/B would be a different OS; the mkosi+UKI+sysupdate fallback stays unused.

ADR-0046 defines configuration preservation and persistent daemon state compatibility so that sharing `/var` does not prevent rollback.
