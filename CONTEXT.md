# FWOS

Open-source firewall and router appliance (working name) for the homelab/SMB edge. The first-class appliance is a VM; not a low-power or small-resource device; 10G is a design floor.

## Language

**Host image**:
The bootable appliance operating system: a Fedora bootc remix plus overlay and branding.
_Avoid_: Host, OS image, appliance image

**Built-in addon**:
An OCI image FWOS ships as part of the product (our binaries or wrapped vendor daemons).
_Avoid_: Jail image, plugin, appliance addon

**Addon**:
A third-party OCI image installed on the appliance.
_Avoid_: Plugin, app

**Workstation tooling**:
The CLI used on the Fedora build machine to build the host image and run a guest. It is not installed on the appliance.
_Avoid_: Host, fwos-dev (the likely git remote, not the concept)

**Host program**:
A first-party binary shipped in the host image `/usr`, not as an OCI image. Not a Fedora rescue tool (`ip`, `nft`, `ethtool`).
_Avoid_: Host package, daemon on the host (those may be Built-in addons)

**netd**:
The built-in addon that is the appliance config API. It applies Desired state in the Forwarding netns (nft, addresses, routes, WireGuard, NIC placement) and generates config for Kea and Unbound. In v1 it is the only process with `CAP_NET_ADMIN`. UI and addons talk to it over one unix socket.
_Avoid_: firewalld, host netd

**Desired state**:
The appliance configuration `netd` applies: interfaces, addresses, VLANs, firewall, static routes, DHCP, DNS, WireGuard, and NIC placement. TOML on `/var`; JSON on the unix socket. Same types.
_Avoid_: running config, candidate config, appliance config

**Appliance CLI**:
A first-party client of `netd`'s unix socket, used on the appliance. Not Workstation tooling.
_Avoid_: fwos-dev, host CLI

**UI**:
A built-in addon that is a client of `netd`'s unix socket. It runs in the Management netns.
_Avoid_: WUI, web GUI, the API (that is netd)

**Host netns**:
The appliance's initial network namespace (PID 1). It is emptied of Traffic NICs and Management NICs. Addon manifest `mgmt` does **not** mean this namespace.
_Avoid_: mgmt netns, init netns

**Management netns**:
The network namespace that owns the Management NIC, SSH, the UI, and the Appliance CLI. On-box and addon-manifest name: `mgmt`.
_Avoid_: Host netns (for this role), admin netns

**Forwarding netns**:
The network namespace that owns Traffic NICs, the VLANs used for forwarding, and the in-kernel routing, firewall, conntrack, qdisc, and WireGuard data plane. On-box name: `fwd`. Addon manifest `fwd` means this namespace.
_Avoid_: Data-plane netns, router netns

**Private netns**:
An addon's own empty network namespace: no NIC, unix sockets only. Addon manifest `none` means this.
_Avoid_: Isolated netns, none netns

**Traffic NIC**:
An interface whose packets are routed or firewalled. It is moved into the Forwarding netns. Includes virtio-net, physical NICs, and VFs.
_Avoid_: Data NIC, LAN/WAN NIC (roles, not the class)

**Management NIC**:
An interface used only to reach SSH and the UI. It is moved into the Management netns.
_Avoid_: Admin NIC
