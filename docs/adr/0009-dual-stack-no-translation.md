# Dual-stack capable, no mixed-family translation in v1

Each WAN/LAN interface can be IPv4-only, IPv6-only, or both. IPv4 forwarding uses NAT44 when a v4 WAN exists. IPv6 is routed (RA + DHCPv6-PD onto the LAN when the WAN has PD). NAT64, NAT46, NAT66, and NPTv6 are out of v1: a v4-only LAN and a v6-only WAN do not interoperate until that is a product.

Considered: shipping NAT64/464XLAT in v1 so “IPv6-only on either side” still reaches the other family. Rejected — that is a translation product, not an addressing checkbox.
