# Built-in addons run as rootful systemd Quadlets + crun

`netd`, Kea, Unbound, and the UI are OCI images started by systemd Quadlets with crun. They join an existing netns via `NetworkNamespacePath=` (`fwd` or `mgmt`) or use a private netns (`none`). No CNI, no Docker bridge, no extra hop. The `netd` unix socket is a pathname socket on a `/var` bind-mount (filesystem, not netns).

Considered: systemd-nspawn as the jail runtime. Rejected — addons are already OCI; Quadlet/crun is the Fedora bootc runner that understands that.
