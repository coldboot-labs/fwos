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
The CLI used on the Fedora build machine to build the host image and run a guest. It is not installed on the appliance; v1 guests are observed through the Appliance console on serial and the UI over HTTPS, without SSH.
_Avoid_: Host, fwos-dev (the likely git remote, not the concept)

**Host program**:
A first-party binary shipped in the host image `/usr`, not as an OCI image. Not a Fedora rescue tool (`ip`, `nft`, `ethtool`).
_Avoid_: Host package, daemon on the host (those may be Built-in addons)

**Network startup program**:
The Host program that prepares the network namespaces, their fixed links, and Traffic NIC placement before `netd` starts. It exits after setup; `netd` owns ongoing network policy.
_Avoid_: host netd, network manager, netns oneshot

**netd**:
The built-in addon that owns Desired state and ongoing network policy, including configuration of Kea and Unbound. It operates in the Forwarding netns and serves the UI, Appliance CLI, and Addons.
_Avoid_: firewalld, host netd

**Desired state**:
The network configuration `netd` applies: interfaces, addresses, VLANs, firewall, static routes, DHCP, DNS, WireGuard, interface roles, UI exposure, and Apply confirmation. It excludes Identity configuration, which controls operator access.
_Avoid_: running config, candidate config, appliance config

**Draft Desired state**:
An administrator's private proposed revision of Desired state, based on an Accepted Desired state revision, which can contain related edits across UI pages. Saving a draft does not change the live configuration or make it accepted.
_Avoid_: running config, accepted configuration

**Accepted Desired state**:
A revision of Desired state that has been successfully applied and, when Apply confirmation is required, confirmed by an administrator. The latest accepted revision is the normal recovery target; its predecessor remains available for manual restoration.
_Avoid_: last good config, committed config, saved config

**Apply confirmation**:
An optional administrator acknowledgement required to accept newly applied Desired state. If its deadline expires without acknowledgement, the previous Accepted Desired state is restored.
_Avoid_: Host update acknowledgement, boot health check

**Authentication source**:
A configured source that verifies an operator's identity, such as FWOS local accounts or a future external identity source. Several sources may coexist without merging identities that happen to use the same username.
_Avoid_: user database (for every source), authorization backend

**Identity configuration**:
The appliance's administrator accounts and credentials, together with its configured Authentication sources and authentication requirements. It is separate from network Desired state and is not reverted by network recovery.
_Avoid_: Desired state (for operator identities), network secrets (for administrator credentials)

**Appliance console**:
The Host program that provides the operator interface on VGA and serial, including the Bootstrap console and authenticated recovery after Bootstrap. It does not provide a Host shell or full configuration commands in v1.
_Avoid_: Appliance CLI (for the v1 console), getty, rescue shell

**Appliance CLI**:
The full command-line interface for post-bootstrap configuration, deferred to v2. It is a client of `netd` and the Host update program, distinct from the v1 Appliance console and Workstation tooling.
_Avoid_: fwos-dev, host CLI, rescue shell, bash, login shell

**UI**:
The built-in addon that is the primary operator interface for Bootstrap, configuration, status, import/export, and Host update in v1. It lives in the Management netns and uses `netd` for Desired state and the Host update program for Host update.
_Avoid_: WUI, web GUI, the API (that is netd), Python UI, a second public API besides the UI's HTTPS

**Bootstrap**:
The first-boot procedure that establishes appliance ownership by creating the first administrator and applying interface roles and UI exposure. Until it completes, first-boot reachability applies and an incomplete attempt may be discarded and repeated.
_Avoid_: setup wizard, initial config (that is Desired state via the UI)

**Bootstrap console**:
The unauthenticated first-boot mode of the Appliance console that lists Traffic NICs and opts exactly one into temporary addressing so the Bootstrap UI is reachable. This temporary reachability is separate from Desired state and lasts until Bootstrap completes or the operator replaces it.
_Avoid_: rescue shell, Anaconda, login, first-boot wizard (that is the UI)

**Host update program**:
The Host program that owns Host update operations. Its operator clients are the UI in v1 and the Appliance CLI in v2; Host updates are separate from `netd`'s ownership of Desired state.
_Avoid_: bootc (the mechanism), fwupd, update daemon, netd

**Host netns**:
The appliance's initial network namespace (PID 1). Traffic NICs leave it at boot. Addon manifest `mgmt` does **not** mean this namespace.
_Avoid_: mgmt netns, init netns

**Management netns**:
The network namespace that owns the UI; its on-box and addon-manifest name is `mgmt`. Physical NICs do not live here, and v1 has no network SSH.
_Avoid_: Host netns (for this role), admin netns, SSH netns

**Forwarding netns**:
The network namespace that owns Traffic NICs, the VLANs used for forwarding, and the in-kernel routing, firewall, conntrack, qdisc, and WireGuard data plane. On-box name: `fwd`. Addon manifest `fwd` means this namespace.
_Avoid_: Data-plane netns, router netns

**Private netns**:
An addon's own empty network namespace: no NIC, unix sockets only. Addon manifest `none` means this.
_Avoid_: Isolated netns, none netns

**Traffic NIC**:
An interface whose packets are routed or firewalled, including a Management NIC. It is moved into the Forwarding netns at boot. Includes virtio-net, physical NICs, and VFs.
_Avoid_: Data NIC, LAN/WAN NIC (roles, not the class)

**WAN**:
An L2 role on a Traffic NIC (untagged) or a VLAN on one: the upstream. v1 never exposes the UI on it. A WAN and a LAN must not share the same parent and tag.
_Avoid_: WAN NIC (a Traffic NIC may carry a WAN)

**LAN**:
An L2 role on a Traffic NIC (untagged) or a VLAN on one: a downstream the appliance routes. DHCP and DNS are LAN services. When the UI is exposed on a LAN, the operator-facing address is that LAN's interface address. A WAN and a LAN must not share the same parent and tag.
_Avoid_: LAN NIC

**Management NIC**:
A Traffic NIC used only to reach the UI. It stays in the Forwarding netns and owns the whole parent: no WAN or LAN on that NIC. It is not a LAN: no DHCP or DNS, and it does not forward to WAN or LANs. The operator sets an on-link static address; there is no gateway. v1 does not use it for SSH. It is always in the UI exposure set.
_Avoid_: Admin NIC, SSH NIC, dedicated management port

**UI exposure**:
The non-WAN interfaces on which the UI is reachable. Every Management NIC is included. LAN membership is chosen by the operator. The operator-facing address is each exposed interface's own address.
_Avoid_: mgmt VLAN, stick exception, Management NIC placement

**bootc deployment**:
A bootable copy of the Host image. Two exist at a time: the running one and the previous one. They share `/var`.
_Avoid_: slot, A/B partition, dual root, deployment slot

**Release**:
A Host image tag: the Host image and the Built-in addons embedded in it. Third-party Addons are not part of a Release.
_Avoid_: system upgrade, appliance version, firmware bundle

**Disk image**:
A prebuilt virtual disk of the Host image. Attaching and booting it is the KVM-first First install. Not the Host image itself. A published Disk image has no injected admin credential and v1 tests do not inject an SSH key. Guests are observed on serial and HTTPS.
_Avoid_: appliance image, OS image, qcow2 (the format, not the concept)

**Installer**:
A self-contained bootable ISO that writes the Host image onto a disk the operator has selected and approved for wipe. One Installer serves metal and a VM with an empty disk. It embeds the Host image; First install does not pull a registry. It does not create the admin or hostname; that is Bootstrap.
_Avoid_: live image, live USB, Anaconda (the mechanism, not the concept), setup wizard (that is Bootstrap)

**Host disk layout**:
The fixed whole-disk layout both First-install paths write: firmware boot partition plus one root that holds both bootc deployments and `/var`. On the Installer, the operator chooses which non-removable disk receives it and must approve the wipe. Removable media are not a First-install target. Not a customizable partition scheme.
_Avoid_: partitioning, dual-boot, A/B partitions, keep-`/var` reinstall

**First install**:
The first write of the Host image onto a machine's disk, from a Disk image or an Installer.
_Avoid_: deploy, provision, image (as a verb)

**Host update**:
Replacing the running bootc deployment with the Host image from a newer Release. Reboot is a separate operator step. The previous bootc deployment remains the rollback target.
_Avoid_: system upgrade, dnf update, ostree upgrade
