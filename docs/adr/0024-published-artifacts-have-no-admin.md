# Published Disk image and Installer have no pre-set admin

A published Disk image or Installer ships no injected SSH key and no default password. Console/serial tells the operator how to reach the bootstrap UI; SSH exists after bootstrap. Workstation tooling may still inject a key into a guest disk used only as a test seam. cloud-init/NoCloud is not a v1 product dependency. A well-known first-boot password is not shipped.
