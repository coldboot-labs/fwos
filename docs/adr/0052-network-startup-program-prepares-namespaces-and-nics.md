# Network startup program prepares namespaces and NICs before netd

The existing startup Host program becomes the Network startup program: it creates `fwd` and `mgmt`, prepares their fixed virtual links and initial plumbing with the Host netns, and moves Traffic NICs into `fwd` before `netd` starts. It temporarily has the network administration privileges needed for that work and exits after setup, while `netd` remains the owner of ongoing routing and firewall policy. This accepts a limited startup privilege exception to simplify preparation of the environment in which `netd` runs.

Amends ADR-0004, ADR-0007, ADR-0008, and ADR-0038. The three namespaces in ADR-0003 and the UI's placement in `mgmt` stand.
