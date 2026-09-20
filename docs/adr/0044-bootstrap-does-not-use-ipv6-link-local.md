# Bootstrap does not use IPv6 link-local addresses

v1 does not support reaching the Bootstrap UI through IPv6 link-local addresses, because the DNAT path from `fwd` to `mgmt` cannot forward a link-local client's source address onto the veth link. IPv6 Bootstrap uses ULA addressing on the console-opted Traffic NIC; ADR-0049 subsequently restricts IPv4 Bootstrap to RFC1918 addresses. The UI stays in `mgmt`, and dual-stack operation remains supported; extra local handling solely to preserve IPv6 link-local Bootstrap is outside v1.

Revises the permitted-address policy in ADR-0028. The console-gated overlay in ADR-0039 stands.
