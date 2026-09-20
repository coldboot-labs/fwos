# netd is the appliance config API

`netd` owns Desired state on `/var`: interfaces, addresses, VLANs, nft, static routes, DHCP, DNS, WireGuard, interface roles, and UI exposure. It programs nft/addresses/WG itself and **generates** Kea and Unbound config. The UI and addons have one unix socket. Kea and Unbound have no public API on the box.

Considered: `netd` as nft-only with the UI fanning out to Kea/Unbound. Rejected — that makes vendor daemons the product contract.

Amended by ADR-0043: operator changes to Desired state are validated before mutation, and a failed apply restores the previous accepted configuration.

ADR-0059 uses the same revision-based validation and application engine for the UI's normal Review and apply workflow and its standalone Save and apply shortcut.
