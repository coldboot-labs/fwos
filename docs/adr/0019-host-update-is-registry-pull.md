# Host update is a registry pull

v1 Host update obtains the next Release by pulling from a container registry (`bootc upgrade` / `switch`). Offline USB/`oci-archive` import and “boot the Installer again to upgrade” are not v1 paths. The temporary update worker uses `fwd` networking under ADR-0060 so pulling does not depend on the Host-to-`mgmt` route or extra upstream address space for internal namespaces.

ADR-0054 requires this update egress path to work with an IPv6-only WAN and no delegated prefix.
