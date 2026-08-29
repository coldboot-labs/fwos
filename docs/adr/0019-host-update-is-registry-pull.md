# Host update is a registry pull

v1 Host update obtains the next Release by pulling from a container registry (`bootc upgrade` / `switch`). Offline USB/`oci-archive` import and “boot the Installer again to upgrade” are not v1 paths. The Host netns veth to `mgmt` (ADR-0003) exists so that pull can work after NICs have left the Host netns.
