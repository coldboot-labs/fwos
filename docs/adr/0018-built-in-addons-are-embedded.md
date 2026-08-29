# Built-in addons are embedded in the Host image

Built-in addon OCI images are copied into the Host image at build. A Release is that Host image. `bootc upgrade` plus reboot moves `/usr` and the builtins together. Pulling builtins from a registry at runtime would make ADR-0016 a policy document, and First install would need a registry. Third-party Addons still live on `/var` and are not in the ostree.
