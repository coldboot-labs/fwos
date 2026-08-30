# Published Disk image and Installer have no pre-set admin

A published Disk image or Installer ships no injected SSH key and no default password. Console/serial tells the operator how to reach the bootstrap UI. v1 has no network SSH at any phase, including tests (ADR-0032). Workstation tooling does not inject a key. cloud-init/NoCloud is not a v1 product dependency. A well-known first-boot password is not shipped.
