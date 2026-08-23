# netd is the appliance config API

`netd` owns Desired state on `/var`: interfaces, addresses, VLANs, nft, static routes, DHCP, DNS, WireGuard, NIC placement, and the mgmt exception path. It programs nft/addresses/WG itself and **generates** Kea and Unbound config. The UI and addons have one unix socket. Kea and Unbound have no public API on the box.

Considered: `netd` as nft-only with the UI fanning out to Kea/Unbound. Rejected — that makes vendor daemons the product contract.
