# netd owns ongoing network administration in v1

v1 routing is static routes via `netd`, which owns ongoing routing and firewall policy. The Network startup program temporarily has `CAP_NET_ADMIN` for initial namespace plumbing and NIC placement, then exits before `netd` starts (ADR-0052). FRR is later: two independent ongoing network policy writers in `fwd` would split nft and routes, so when OSPF/BGP is a product, reopen whether FRR talks to `netd` or gets the cap.

ADR-0052 supersedes absolute capability exclusivity with a startup exception; ongoing policy ownership stays with `netd`.
