# Bootstrap UI uses RFC1918 or ULA addresses

The Bootstrap UI is reachable only through RFC1918 IPv4 or IPv6 ULA addresses on the console-opted Traffic NIC. IPv4 link-local access is also excluded because the routed DNAT path into `mgmt` would forward link-local traffic, contrary to RFC 3927; v1 keeps this path without adding a local relay solely for link-local UI access. This restricts Bootstrap UI addressing, not link-local protocol use such as IPv6 neighbor discovery.

Revises ADR-0028 and completes the address restriction begun in ADR-0044. The console-gated overlay and UI placement in ADR-0039 stand.
