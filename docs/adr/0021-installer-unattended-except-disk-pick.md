# Installer is unattended except picking a disk when there is more than one

The Installer writes the Host image onto one disk and reboots. If there is exactly one writable disk, it asks nothing. If there is more than one, it only asks which disk to wipe. The chosen disk is always wiped; there is no keep-`/var` reinstall. It is not a Fedora installer: no Anaconda partitioning, user creation, timezone, or package screens. Identity is the bootstrap UI after First install. The Installer is not an updater.
