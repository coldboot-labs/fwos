# Kea and Unbound are built-in addons with generated config

v1 DHCP is Kea (DHCPv4 and DHCPv6) in `fwd`. v1 DNS is Unbound in `fwd`. `netd` writes their config; they are not a combined dnsmasq. IPv6 RA is part of address/prefix Desired state (`netd`), not a separate radvd product in v1.
