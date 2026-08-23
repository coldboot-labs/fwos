# Only netd has CAP_NET_ADMIN in v1

v1 routing is static routes via `netd`. FRR is later. Two `CAP_NET_ADMIN` processes in `fwd` would split nft and routes. When OSPF/BGP is a product, reopen whether FRR talks to `netd` or gets the cap.
