# FWOS

Open-source firewall and router appliance (working name) for the homelab/SMB edge. The first-class appliance is a VM; not a low-power or small-resource device; 10G is a design floor.

## Language

**Host image**:
The bootable appliance operating system: a Fedora bootc remix plus overlay and branding.
_Avoid_: Host, OS image, appliance image

**Built-in addon**:
An OCI image FWOS ships as part of the product (our binaries or wrapped vendor daemons). It is embedded in the Host image, not pulled at runtime.
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
A first-party client on the appliance of `netd`'s unix socket and of the Host update program's unix socket. Not Workstation tooling.
_Avoid_: fwos-dev, host CLI

**UI**:
A built-in addon that is a client of `netd`'s unix socket. v1 is bootstrap-only and does not call the Host update program; later it will. Before bootstrap it listens in the Host netns over HTTPS; steady state is the Management netns.
_Avoid_: WUI, web GUI, the API (that is netd)

**Bootstrap**:
The first-boot procedure that creates the admin and applies NIC placement (Management NIC or stick exception). Until it completes, first-boot reachability applies.
_Avoid_: setup wizard, initial config (that is Desired state via the UI)

**Bootstrap console**:
The Host program on VGA and serial, before bootstrap, that lists Host-netns NICs and sets ephemeral addressing (static, DHCP, or SLAAC) so an operator can reach the UI. No login. Not Appliance CLI, not `netd`. Gone after bootstrap.
_Avoid_: rescue shell, Anaconda, login, first-boot wizard (that is the UI)

**Host update program**:
The Host program in the Host netns that wraps `bootc`. Appliance CLI and later the UI talk to it over a unix socket on `/var`, not via `netd`.
_Avoid_: bootc (the mechanism), fwupd, update daemon, netd

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

**bootc deployment**:
A bootable copy of the Host image. Two exist at a time: the running one and the previous one. They share `/var`.
_Avoid_: slot, A/B partition, dual root, deployment slot

**Release**:
A Host image tag: the Host image and the Built-in addons embedded in it. Third-party Addons are not part of a Release.
_Avoid_: system upgrade, appliance version, firmware bundle

**Disk image**:
A prebuilt virtual disk of the Host image. Attaching and booting it is the KVM-first First install. Not the Host image itself. A published Disk image has no injected admin credential; Workstation tooling may build a separate guest disk with an injected SSH key for tests.
_Avoid_: appliance image, OS image, qcow2 (the format, not the concept)

**Installer**:
A self-contained bootable ISO that writes the Host image onto a disk. One Installer serves metal and a VM with an empty disk. It embeds the Host image; First install does not pull a registry.
_Avoid_: live image, live USB, Anaconda (the mechanism, not the concept)

**First install**:
The first write of the Host image onto a machine's disk, from a Disk image or an Installer.
_Avoid_: deploy, provision, image (as a verb)

**Host update**:
Replacing the running bootc deployment with the Host image from a newer Release. Reboot is a separate operator step. The previous bootc deployment remains the rollback target.
_Avoid_: system upgrade, dnf update, ostree upgrade
