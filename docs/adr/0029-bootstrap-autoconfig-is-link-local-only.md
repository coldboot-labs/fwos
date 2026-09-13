# No automatic DHCP during bootstrap; SLAAC is allowed

Until the Bootstrap console opts a NIC into DHCP, Traffic NICs do not run DHCPv4 or DHCPv6 (including PD). Accidental DHCP on a NIC already plugged into an ISP can consume a lease some ISPs are slow to reissue on the later WAN. Un-opted NICs also do not run SLAAC/RA (ADR-0039).

The UI still does not bind to GUA or other global addresses during first-boot (ADR-0028). The Bootstrap console is how the operator starts DHCP or sets a static address on the single opted NIC.

Revised by ADR-0039.
