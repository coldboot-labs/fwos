# Host disk layout root is ext4

The Disk image and the Installer write one ext4 `/` (firmware boot plus that root). bootc deployments are the rollback story (ADR-0015); btrfs snapshots or extra subvolumes on that root would be a second updater, and the Disk image builder cannot create btrfs subvolumes anyway. Considered: switch `/` to btrfs for “options later” (compression, send/receive, snapper-style layout). Rejected — those are not benefits on a bootc root, and both First-install paths must write the same Host disk layout.
