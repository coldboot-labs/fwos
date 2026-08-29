# No automatic DHCP during bootstrap; SLAAC is allowed

Until the Bootstrap console opts a NIC into DHCP, Host-netns NICs do not run DHCPv4 or DHCPv6 (including PD). Accidental DHCP on a NIC already plugged into an ISP can consume a lease some ISPs are slow to reissue on the later WAN.

IPv6 link-local, IPv4 link-local (`169.254/16`), and SLAAC/RA are allowed on every Host-netns NIC. The UI still does not bind to GUA or other global addresses (ADR-0028). The Bootstrap console is how the operator starts DHCP or sets a static address.
