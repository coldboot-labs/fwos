# FWOS

Open-source firewall and router appliance (working name).

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
