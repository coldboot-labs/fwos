# How comparable appliances and bootc products split git repositories

Question: how do comparable open-source appliances and bootc / image-mode products split (or refuse to split) git repositories? For each, record what lives in the image/host repo vs app repos, how artifacts flow at build time, and whether issues live in a hub repo.

Primary sources only. Claims cite the owning project's docs, READMEs, or first-party APIs.

## Patterns that matter

Four grouping patterns show up repeatedly:

1. **Host image vs optional apps, with a glue/build repo.** OPNsense, OpenWrt, Turris OS, Home Assistant OS, TrueNAS. The bootable image is assembled from a small number of first-party trees; optional software is a catalog that the running system installs later (packages, OCI images, or compose templates).
2. **Refuse to split.** IPFire 2.x keeps the Linux From Scratch tree, WUI, and add-on recipes in one git repository. First-party code is not a constellation of GitHub remotes.
3. **Layered bootc: one git repo per derived image.** Fedora bootc publishes a binary base image; Universal Blue and Bluefin each keep a Containerfile repo that `FROM`s an upstream bootc image. Workload containers are not the host image; they are listed, Quadlet-embedded, or pulled at runtime. First-party RPMs often live in a separate `packages` repo feeding COPR.
4. **Do not over-split first-party appliance code.** VyOS 1.1 split “into way too many submodules”; 1.2+ consolidates rewritten code into `vyos-1x`. The image builder stays a separate repo. That is the closest published regret note.

Issue-tracker topology is independent of git topology:

- **Hub tracker, even when git is split:** IPFire Bugzilla, VyOS Phabricator, TrueNAS Jira (OS), OpenWrt Bugzilla (core).
- **Issues follow the git repo:** OPNsense (`core` vs `plugins`), Home Assistant (`operating-system`, `supervisor`, `addons`, `core`), Universal Blue product repos, TrueNAS Apps GitHub, Turris GitLab per repo.
- **Architecture RFC hub beside per-repo bugs:** Home Assistant `architecture` discussions; Fedora bootc / CoreOS tracker issues.

FWOS already decided that `fwos` is a meta hub and that the bootc image tree does not live there. The comparables that match that split are Home Assistant OS, Fedora/ublue bootc layering, OPNsense `tools`+`src` vs `core`/`plugins`, OpenWrt core vs feeds, and TrueNAS middleware vs apps catalog. The comparables that argue against further splitting first-party daemons/UI are VyOS's consolidation and IPFire's single tree.

---

## OPNsense

### Repositories

The official architecture doc lists four git parts of the stack, plus plugins as of 16.1:

| Repo | Owns | URL |
|------|------|-----|
| `src` | FreeBSD-derived operating system | <https://github.com/opnsense/src> |
| `ports` | Third-party software | <https://github.com/opnsense/ports> |
| `core` | GUI and system configuration | <https://github.com/opnsense/core> |
| `tools` | Shell glue that produces images | <https://github.com/opnsense/tools> |
| `plugins` | Optional extensions | <https://github.com/opnsense/plugins> |

Source: [Architecture — Core system](https://docs.opnsense.org/development/architecture.html), [Development workflow](https://docs.opnsense.org/development/workflow.html).

`core` is **not** in `src` because it depends on ports packages that do not belong in the FreeBSD base. It is **not** in `ports` either: the tools repo treats `core` as a package that depends on the ports it needs, so on images “the core code is just another package.” That makes upgrading GUI/config fast without touching base. ([Development workflow](https://docs.opnsense.org/development/workflow.html))

`plugins` is a collection repository: one source directory per plugin; “the core can be thought of a lone plugin.” Plugins can add GUI pages, themes, authentication methods, devices, extra packages, backend tasks, boot scripts, and `loader.conf` bits. ([plugins/README.md](https://github.com/opnsense/plugins/blob/master/README.md))

Related repos used at build/runtime, not the host/app split: `update` (bootstrap/reinstall a running system; [workflow](https://docs.opnsense.org/development/workflow.html)), `docs`, `changelog`. The org lists 22 public repositories ([opnsense on GitHub](https://github.com/opnsense)).

### What lives in the image vs apps

The DVD/memstick/nano/vm **image** is assembled from five build stages: base, kernel, ports, plugins, and core. Plugins and core become packages in the firmware set; they are not a separate OCI runtime. ([tools/README.md](https://github.com/opnsense/tools/blob/master/README.md))

On a running box, plugins are optional `pkg` packages shown in **System → Firmware → Plugins**. Community (lower-tier) plugins are hidden until the operator checks “Show community plugins.” ([Firmware](https://docs.opnsense.org/manual/firmware.html))

### Artifact flow at build time

`tools` is designed to run on stock FreeBSD using chroot, not by mutating the build host. Clone `tools`, `make update` (pulls `core`, `plugins`, `ports`, `src`, `tools`), then `make base`, `make kernel`, `make ports`, `make plugins`, `make core`, then `make dvd` / `serial` / `vga` / `nano` / `vm`. Package sets are signed if keys exist. Nightly composite is `make nightly`. ([tools/README.md](https://github.com/opnsense/tools/blob/master/README.md), [workflow](https://docs.opnsense.org/development/workflow.html))

Developer loop for GUI work: clone `core` on a running OPNsense VM and `make mount` overlays the repo on `/usr/local`. ([workflow](https://docs.opnsense.org/development/workflow.html))

### Issues

GitHub Issues per code repo, not a single hub. `core` CONTRIBUTING tells reporters to search that repo's issues; features “beyond the scope of OPNsense may still be provided using the plugin framework” and points at <https://github.com/opnsense/plugins/issues>. ([core/CONTRIBUTING.md](https://github.com/opnsense/core/blob/master/CONTRIBUTING.md)) `plugins` README likewise: “open GitHub issue to get in touch.” ([plugins/README.md](https://github.com/opnsense/plugins/blob/master/README.md))

---

## IPFire

IPFire publishes git layout. Official source hosting is [git.ipfire.org](https://git.ipfire.org/) / [cgit.ipfire.org](https://cgit.ipfire.org/), not a GitHub org as the canonical remote. ([Source Code](https://www.ipfire.org/docs/devel/sources))

### Repositories

| Tree | Role |
|------|------|
| `ipfire-2.x.git` | Production 2.x distribution: LFS build, WUI, Pakfire, add-on recipes in one clone |
| `ipfire-3.x.git` | 3.x package recipes (incomplete; each developer also has a people sandbox) |
| `pakfire.git` | “package management and build system for IPFire 3.x” (cgit description on that tree) |

2.x branches: `master` (Stable / shipped), `next` (Testing / next Core Update), `unstable` (nightly). Clone:

```
git clone https://git.ipfire.org/pub/git/ipfire-2.x.git
```

([Source Code](https://www.ipfire.org/docs/devel/sources), [Git Guide](https://www.ipfire.org/docs/devel/git))

The 2.x tree is a Linux From Scratch style layout (`lfs/`, `src/`, `config/`, `html/`, `make.sh`). A GitHub **mirror** exists at <https://github.com/ipfire/ipfire-2.x> (same tree: config, html, langs, lfs, src, tools, make.sh). Official clone URLs remain git.ipfire.org.

### What lives in the image vs apps

**Refuse to split for 2.x.** The same repository “contains the source code of IPFire 2.x which is used to build the whole distribution from scratch.” ([ipfire-2.x README on the GitHub mirror](https://github.com/ipfire/ipfire-2.x), as indexed from the project's published tree)

Add-ons are still a runtime concept: Pakfire installs `.ipfire` packages from `pakfire.ipfire.org` (see `src/pakfire/pakfire.conf` in the 2.x tree). Recipes for those add-ons live **in the same git tree**: write `lfs/<name>`, add `lfsmake2 <name>` to `make.sh`, put rootfiles in `config/rootfiles/packages/`, optionally copy `src/paks/default` to `src/paks/<name>` for install/uninstall/update scripts. ([Building Add-ons](https://www.ipfire.org/docs/devel/ipfire-2-x/addon-howto), [Pakfire](https://www.ipfire.org/docs/configuration/ipfire/pakfire))

3.x is more “distro of package recipes in one git tree” plus per-developer sandboxes, not one repo per addon. ([Source Code](https://www.ipfire.org/docs/devel/sources), [Development](https://www.ipfire.org/docs/devel))

### Artifact flow at build time

`./make.sh downloadsrc` fetches upstream tarballs into `cache/`; `./make.sh gettoolchain` fetches stage 1; `./make.sh build` produces ISO and disk images in the repo root (`ipfire-*.iso`, `ipfire-*.img.xz`). Add-on `.ipfire` files land under `packages/`. There is no separate image repo consuming artifacts from app repos. ([Build Howto](https://www.ipfire.org/docs/devel/ipfire-2-x/build-howto), [Building Add-ons](https://www.ipfire.org/docs/devel/ipfire-2-x/addon-howto))

Patches go to `development@lists.ipfire.org` / Patchwork, not GitHub PRs as the primary path. ([Build Howto](https://www.ipfire.org/docs/devel/ipfire-2-x/build-howto), [Submitting Patches](https://www.ipfire.org/docs/devel/submit-patches))

### Issues

**Hub tracker:** [Bugzilla](https://bugzilla.ipfire.org/). Official policy: do not report bugs on the support forums or mailing lists; “open a bug report on the bugtracker.” ([Bugzilla](https://www.ipfire.org/docs/devel/bugzilla))

---

## Home Assistant OS vs Supervisor vs Apps

Home Assistant is the cleanest published split of **host image / orchestrator / apps**, with Core as a fourth git repo.

### Architecture

Developer architecture overview:

- Operating System: “bare minimal Linux environment to run Supervisor and Core.”
- Supervisor: manages the OS (and Core, backups, apps).
- Core: the home-automation application.

([Architecture overview](https://developers.home-assistant.io/docs/architecture_index), [Supervisor](https://developers.home-assistant.io/docs/supervisor))

OS README: Docker is the container engine; the OS deploys Supervisor; Supervisor runs Core and Apps in separate containers. HAOS is **Buildroot**, not Ubuntu. Updates use RAUC. ([operating-system/README.md](https://github.com/home-assistant/operating-system/blob/dev/README.md), [OS intro](https://developers.home-assistant.io/docs/operating-system))

### Repositories

| Repo | Owns | URL |
|------|------|-----|
| `operating-system` | Buildroot br2-external, board configs, GH Actions, helper scripts | <https://github.com/home-assistant/operating-system/> |
| `buildroot` (fork) | Buildroot itself (aim: keep patches minimal) | <https://github.com/home-assistant/buildroot/> |
| `supervisor` | Orchestrator API, Core/OS/app lifecycle | <https://github.com/home-assistant/supervisor> |
| `core` | Home Assistant Core | <https://github.com/home-assistant/core> |
| `addons` | Official Apps (formerly add-ons) catalog | <https://github.com/home-assistant/addons> |
| `docker-base` | Alpine/Debian/Ubuntu/Python bases for Core and Apps | <https://github.com/home-assistant/docker-base> |
| `builder` | GitHub Actions that build multi-arch app images | <https://github.com/home-assistant/builder> |
| `version` | Channel pins: which Supervisor, Core, OS, helper images | <https://github.com/home-assistant/version> |
| `architecture` | RFC discussions and ADRs | <https://github.com/home-assistant/architecture> |
| `addons-example` / `apps-example` | Template for third-party app repos | linked from [Developing an app](https://developers.home-assistant.io/docs/apps) |
| `hassio-addons` (community org) | Community apps | <https://github.com/hassio-addons> |

OS clone uses a git submodule for Buildroot. ([Getting started](https://developers.home-assistant.io/docs/operating-system/getting-started))

### What lives in the image vs apps

**Host image (`operating-system`):** kernel, bootloader (GRUB or U-Boot), systemd, Docker Engine, AppArmor, RAUC slots, SquashFS. It does **not** vendor Core or Apps into the rootfs as first-class packages. Supervisor is the default container the OS starts. ([OS README](https://github.com/home-assistant/operating-system/blob/dev/README.md))

**Apps:** OCI images in a registry. A git “app repository” can contain many apps (one folder each) plus `repository.yaml`. Official apps (Mosquitto, MariaDB, Samba, SSH, Z-Wave JS, …) live in `home-assistant/addons`. Third parties publish their own git remotes; users paste the URL into the Supervisor store. ([Developing an app](https://developers.home-assistant.io/docs/apps), [Create an app repository](https://developers.home-assistant.io/docs/apps/repository), [addons/README.md](https://github.com/home-assistant/addons/blob/master/README.md))

### Artifact flow at build time

- **OS:** `scripts/enter.sh make <board>` inside a privileged Debian build container; outputs disk images under `output/images/`. Development artifacts: <https://os-artifacts.home-assistant.io/>. PRs target `dev`. ([Getting started](https://developers.home-assistant.io/docs/operating-system/getting-started))
- **Supervisor:** Dockerfile + GH Actions; images `hassio-supervisor` per arch. Release channels: merge to `main` → `dev` image → GitHub Release → `beta` → update `stable.json` → promote that build to `stable`. ([supervisor/README.md](https://github.com/home-assistant/supervisor/blob/main/README.md))
- **Apps:** preferred path is pre-built multi-arch images on GHCR via builder actions; `config.yaml` `image:` field names the registry image. Local-on-device builds exist but are discouraged for established repos. ([Publishing](https://developers.home-assistant.io/docs/apps/publishing))
- **Composition at runtime, not OS build:** `home-assistant/version` `stable.json` pins `supervisor`, `homeassistant` (per machine), `hassos` (per board), helper containers (`cli`, `dns`, `audio`, `multicast`, `observer`), OTA URL template, and image name templates such as `ghcr.io/home-assistant/{arch}-hassio-supervisor`. ([stable.json](https://github.com/home-assistant/version/blob/master/stable.json))

The OS image build does not compile Core or the add-on catalog. Supervisor pulls those images using the version pins.

### Issues

**Per git repo**, plus an architecture hub:

- OS bugs/PRs: `home-assistant/operating-system`
- Supervisor: `home-assistant/supervisor` (significant changes: RFC first; [supervisor README](https://github.com/home-assistant/supervisor/blob/main/README.md))
- Official apps: `home-assistant/addons` issues ([addons README](https://github.com/home-assistant/addons/blob/master/README.md))
- Core product bugs: `home-assistant/core`
- Cross-cutting design: create a discussion in `home-assistant/architecture`; decisions become ADRs in `adr/`. ([architecture/README.md](https://github.com/home-assistant/architecture/blob/master/README.md))

There is no single GitHub issues hub that holds OS + Supervisor + Core + Apps together. The `version` repo is the **release composition** hub, not the issue hub.

---

## Universal Blue / Fedora bootc derived images

bootc itself is a **client**: transactional in-place OS updates from OCI images. It is not a Linux distribution. Start from adopters / a base image, not by forking bootc. ([bootc README](https://github.com/bootc-dev/bootc/blob/main/README.md), [bootc image layout](https://bootc-dev.github.io/bootc/bootc-images.html))

### Fedora bootc base images

Canonical git: [gitlab.com/fedora/bootc/base-images](https://gitlab.com/fedora/bootc/base-images).

That repo **creates the official binary bases** (`quay.io/fedora/fedora-bootc` “standard” tier, plus internal `minimal` / `minimal-plus`). The documented default UX is **not** to fork this repo. Layer on top of the published image, or use the existing image as a builder for a from-scratch content set. ([base-images README](https://gitlab.com/fedora/bootc/base-images/-/blob/main/README.md))

Examples of layered Containerfiles (cloud-init, Quadlet workloads, nvidia, tailscale, …) live in a **separate** repo: [fedora/bootc/examples](https://gitlab.com/fedora/bootc/examples). `embed-workloads` pulls app container images **during** `podman build` and installs Quadlet units under `/etc/containers/systemd/`. ([examples README](https://gitlab.com/fedora/bootc/examples/-/blob/main/README.md), [embed-workloads](https://gitlab.com/fedora/bootc/examples/-/blob/main/embed-workloads/README.md))

Issues for base images: GitLab issues on `fedora/bootc/base-images`. The bootc **client** uses GitHub discussions on `bootc-dev/bootc` / `containers/bootc`. ([bootc README](https://github.com/bootc-dev/bootc/blob/main/README.md))

### Fedora CoreOS config (related ostree/bootc family)

[coreos/fedora-coreos-config](https://github.com/coreos/fedora-coreos-config) is the rpm-ostree **manifest** repo (not a Containerfile). It is explicitly split into reusable layers so derivatives (historically Silverblue etc.) can git-submodule it. Bugs go to **fedora-coreos-tracker**, not this config repo. ([README](https://github.com/coreos/fedora-coreos-config/blob/testing-devel/README.md))

### Universal Blue org layout

ublue-os is “a collection of container images using OCI/Docker containers as a transport … by using bootc.” ([github.com/ublue-os](https://github.com/ublue-os))

| Repo | Owns |
|------|------|
| [ublue-os/main](https://github.com/ublue-os/main) | Common Fedora Atomic bases (base, kinoite, silverblue); intermediate images not used by Aurora/Bazzite/Bluefin are being dropped |
| [ublue-os/bluefin](https://github.com/ublue-os/bluefin), [aurora](https://github.com/ublue-os/aurora), [bazzite](https://github.com/ublue-os/bazzite) | Product **host images** (Containerfile + build_files) |
| [ublue-os/ucore](https://github.com/ublue-os/ucore) | Fedora CoreOS derivative; server/NAS/HCI **host images**; apps run as podman/moby |
| [ublue-os/packages](https://github.com/ublue-os/packages) | RPM spec files → Universal Blue COPRs; not the host image |
| [ublue-os/akmods](https://github.com/ublue-os/akmods) | Out-of-tree kernel modules as container images consumed by product builds |
| [ublue-os/image-template](https://github.com/ublue-os/image-template) | Template for **end-user derived** bootc images |

Bluefin's contributor docs go further: opinionated **workload** files (ujust, GNOME, CLI) moved to [@projectbluefin/common](https://github.com/projectbluefin/common); artwork [@ublue-os/artwork](https://github.com/ublue-os/artwork); Homebrew [@ublue-os/brew](https://github.com/ublue-os/brew). The image repo **assembles** those OCI layers. Graphical apps are Flatpaks from Flathub; Bluefin only keeps ID lists (`flatpaks/*.list`). ([Contributing to Bluefin](https://docs.projectbluefin.io/contributing))

uCore states the CoreOS rule: the root is immutable; **services should run as containers** (podman; docker socket disabled by default). Extra RPMs in the ucore image are host enablement (ZFS, samba, libvirt, cockpit), not the app catalog. ([ucore README](https://github.com/ublue-os/ucore/blob/main/README.md))

### Artifact flow

1. Fedora/CentOS publish `fedora-bootc` / `centos-bootc` / `fedora-coreos` OCI images.
2. ublue `main` (and product repos) `FROM` those, install COPR RPMs from `packages`, copy `system_files`, run `build_files/*.sh`, push to `ghcr.io/ublue-os/<image>`.
3. End users `bootc switch ghcr.io/<user>/<image>` from [image-template](https://github.com/ublue-os/image-template/blob/main/README.md). Disk images (ISO/qcow/raw) are a **separate** GitHub Actions workflow wrapping [bootc-image-builder](https://github.com/osbuild/bootc-image-builder).
4. Workloads: Flatpak lists in the image, Homebrew, Quadlets, or `podman` at runtime — not compiled into `operating-system`-style rootfs packages.

### Issues

**Per product git repo** (`ublue-os/bluefin`, `ucore`, `packages`, `image-template`, …). Bluefin points issues at [issues.projectbluefin.io](https://issues.projectbluefin.io) (GitHub issues on the Bluefin project). Fedora upstream bugs that reproduce on vanilla Atomic go to [fedora-silverblue/issue-tracker](https://github.com/fedora-silverblue/issue-tracker/issues). ([Contributing](https://docs.projectbluefin.io/contributing)) There is no ublue-wide issues hub that owns Aurora+Bazzite+Bluefin+uCore together.

### systemd sysexts (host image stays thin)

[fedora-sysexts](https://github.com/fedora-sysexts) publishes systemd system extensions for Fedora image-mode (bootc and classic ostree). Official Fedora RPMs: [fedora-sysexts/fedora](https://github.com/fedora-sysexts/fedora); community sources: `fedora-sysexts/community`. These are **not** folded into the host git repo; they ship as GitHub Release artifacts consumed by `systemd-sysupdate`. ([fedora README](https://github.com/fedora-sysexts/fedora/blob/main/README.md))

---

## Other firewall / router appliances that keep host image separate from apps

### OpenWrt

OpenWrt's own README: it is **not** a single static firmware; it is a writable root plus `opkg`, and the **main repository uses multiple sub-repositories** (feeds) for package categories. ([openwrt/openwrt README](https://github.com/openwrt/openwrt/blob/master/README.md))

Default feeds (`feeds.conf.default`):

```
src-git packages https://git.openwrt.org/feed/packages.git
src-git luci     https://git.openwrt.org/project/luci.git
src-git routing  https://git.openwrt.org/feed/routing.git
src-git telephony https://git.openwrt.org/feed/telephony.git
src-git video    https://github.com/openwrt/video.git
```

([feeds.conf.default](https://github.com/openwrt/openwrt/blob/master/feeds.conf.default))

**Host/image repo (`openwrt/openwrt`):** toolchain, kernel, target profiles, base packages, image recipes. GitHub is a **mirror** of git.openwrt.org (“not active for check-ins” on the GitHub about text).

**App/UI repos:** `packages` (community ported software), `luci` (web UI as a feed), `routing`, `telephony`, `video`.

Build: `./scripts/feeds update -a && ./scripts/feeds install -a && make menuconfig && make`. Feeds are cloned into the core tree as package definitions; the resulting firmware may include a default package set, and everything else is opkg later. ([README](https://github.com/openwrt/openwrt/blob/master/README.md))

**Issues:** core bugs at [bugs.openwrt.org](https://bugs.openwrt.org) (named in the README “Bug Reports”). Packages feed uses GitHub PRs/issues and its own [CONTRIBUTING.md](https://github.com/openwrt/packages/blob/master/CONTRIBUTING.md). LuCI has its own GitHub repo.

### Turris OS (OpenWrt-based router appliance)

Official docs: “Turris project consists of **more than a hundred repositories**” and tell you which ones to use for issues. ([Most important Turris repositories](https://docs.turris.cz/geek/contributing/repositories/))

| Repo | Owns |
|------|------|
| `os/build` | Build scripts, patches, OpenWrt configuration (Turris-specific fixes that would otherwise go to OpenWrt) |
| `os/packages` | Turris-specific package recipes |
| `os/updater-lists` | Which packages get installed in which situations |
| `updater/updater` | Package manager for unattended updates |
| reForis + Foris Controller (+ per-plugin frontend/backend pairs) | Web UI split core vs plugins |
| Sentinel family | Threat-detection clients/servers, each its own git |

Host image: `os/build` (OpenWrt tree + patches). Apps: `os/packages` plus OpenWrt feeds. UI plugins: **one git repo per plugin** (frontend and backend often split). Issues: file on the repo that owns the code (GitLab primary, GitHub mirrors). That is an extreme split; they had to publish a map so people can find the right remote.

### VyOS (router OS; cautionary split)

VyOS is Debian-based. **Image build** is [vyos/vyos-build](https://github.com/vyos/vyos-build): “top level repository that contains links to repositories with VyOS specific packages (organized as Git submodules)” plus ISO scripts. Packages compile first, then an ISO is built from Debian + first-party debs. ([vyos-build README](https://github.com/vyos/vyos-build/blob/rolling/README.md))

**First-party appliance code** is [vyos/vyos-1x](https://github.com/vyos/vyos-1x): CLI XML, Python conf/op-mode scripts, templates. Explicit history:

> VyOS 1.1.x had its codebase split into way too many submodules for no good reason, which made it hard to navigate or write meaningful changelogs. As the code undergoes rewrite … we consolidate the rewritten code in this package.

([vyos-1x README](https://github.com/vyos/vyos-1x/blob/rolling/README.md))

Development docs: “composed of multiple modules”; “consolidates most packages into vyos-1x”; “Only submit bugfixes in packages other than vyos-1x”; new functionality must use the XML/Python interface. Commits must reference a **Phabricator** task (`T1234`). ([Development](https://docs.vyos.io/en/latest/contributing/development.html))

There is no optional “apps catalog” comparable to HA add-ons or OPNsense plugins: routing daemons (FRR, etc.) are **deb packages baked into the ISO** via `vyos-build/packages/`.

**Issues:** hub is [vyos.dev (Phabricator)](https://vyos.dev/). GitHub is for PRs; GitHub issue counts on `vyos-build` / `vyos-1x` are not the product tracker.

### TrueNAS (NAS appliance; host vs apps)

Not a firewall, but it is an appliance that **keeps the host image separate from apps**.

- **OS / middleware:** [truenas/middleware](https://github.com/truenas/middleware) — “TrueNAS Middleware Source Repo.” Product bugs: **Jira** (<https://jira.ixsystems.com>), not GitHub Issues (README badges: “File Issue” → Jira).
- **ISO/update build:** [truenas/scale-build](https://github.com/truenas/scale-build) historically: `make checkout` pulls source repos, `make packages` builds debs, `make update` / `make iso`. The README now says the live build system “has been moved to an internal infrastructure”; the public tree is historical. Bugs still pointed at Jira.
- **Apps catalog:** [truenas/apps](https://github.com/truenas/apps) — Docker Compose catalog. App developers do **not** put containers in the host image. They add metadata + Jinja templates under `ix-dev/{train}/{app}/`; TrueNAS renders compose at install time and Docker runs the upstream images. Legacy Kubernetes charts lived in [truenas/charts](https://github.com/truenas/charts).

([apps README](https://github.com/truenas/apps/blob/master/README.md), [CONTRIBUTIONS.md](https://github.com/truenas/apps/blob/master/CONTRIBUTIONS.md))

Generated compose files even point bug reports at `https://github.com/truenas/apps`. OS bugs stay on Jira. Two hubs, split by layer.

---

## Comparison

| Project | Host / image git | First-party app/UI git | Optional apps | Build flow | Issue hub? |
|---------|------------------|------------------------|---------------|------------|------------|
| **OPNsense** | `src` + `tools` | `core` (treated as a pkg) | `plugins` collection repo → pkg | `tools` chroot: base, kernel, ports, plugins, core → ISO | GitHub **per repo** (`core` vs `plugins`) |
| **IPFire 2.x** | `ipfire-2.x` **is** the distro | Same tree (`html`, `src`) | Same tree (`lfs` + `src/paks`) → `.ipfire` | `make.sh build` in-tree | **Bugzilla hub** |
| **HA OS** | `operating-system` (+ `buildroot` submodule) | `supervisor`, `core` as OCI | `addons` + third-party git stores as OCI | OS images independent; `version` pins compose at runtime | GitHub **per repo** + `architecture` RFC hub |
| **Fedora bootc** | `fedora/bootc/base-images` (do not fork) | Layered Containerfile elsewhere | Quadlets / podman; optional sysexts | `FROM` official image; examples embed pulls | GitLab issues on base-images; bootc client discussions |
| **Universal Blue** | One git repo per product image | `packages` (RPMs), `common` (workload OCI), Flatpak **lists** | Runtime containers / Flatpaks / Homebrew | GH Actions → GHCR; BIB for disks | GitHub **per product** |
| **OpenWrt** | `openwrt/openwrt` | LuCI is a **feed**, not the image repo | `packages` + other feeds | `feeds update/install` then `make` | Bugzilla for **core**; GitHub for feeds |
| **Turris OS** | `os/build` | reForis/Foris **many** repos | `os/packages` + OpenWrt feeds | OpenWrt-style + updater-lists | GitLab **per repo** (they publish a map) |
| **VyOS** | `vyos-build` (submodules + ISO) | **`vyos-1x` consolidation** | No plugin catalog; debs in the ISO | Build debs then ISO | **Phabricator hub**; GitHub for PRs |
| **TrueNAS** | middleware + (now internal) scale-build | middleware | `truenas/apps` compose catalog | Host debs vs catalog render | **Jira** for OS; GitHub for apps |

---

## Implications for FWOS

Already decided on the map: `fwos` is the meta hub; the bootc image tree must be its own store; daemons/UI/apps may share a git repo. Comparables support that **image vs apps** cut and argue about **how far to split the rest**.

1. **Keep the host-image git repo a Containerfile layer, not a distro of apps.** Fedora bootc and ublue: do not fork `base-images`; `FROM quay.io/fedora/fedora-bootc` (or ublue/uCore) and add overlay, branding, and host RPMs. HAOS is the other pole (purpose-built Buildroot image that only runs Docker + Supervisor). FWOS is bootc, so the ublue/fedora-bootc pole applies.

2. **Optional firewall “apps” should not live in the host-image repo.** OPNsense `plugins`, OpenWrt feeds, HA `addons`, TrueNAS `apps` are the working models. IPFire puts add-on recipes in the same tree because the whole distro is one LFS build — that matches a from-scratch distro, not a Fedora remix.

3. **Do not split first-party daemons/UI into one GitHub repo each without a builder that can stand it.** VyOS documented that 1.1's submodule explosion was “for no good reason” and merged rewritten code into `vyos-1x`. Turris split UI plugins into 100+ remotes and had to write a directory of remotes so issue filers could cope. A **monorepo of first-party apps/UI/daemons** (OPNsense `plugins` as a collection, HA official `addons` as a collection, TrueNAS `ix-dev/community/*`) is the common middle path.

4. **Glue/build vs content.** OPNsense `tools`, OpenWrt core, VyOS `vyos-build`, HA `operating-system`+`version`, ublue product Containerfiles are the assemblers. Content (core GUI, plugins, add-ons) is consumed as packages or OCI. FWOS `fwos` as a hub that does **not** contain the bootc tree matches OPNsense-without-putting-`src`-in-the-hub, or HA's lack of a single super-repo.

5. **Issues can stay on the hub even when git is split.** VyOS (Phabricator), IPFire (Bugzilla), TrueNAS OS (Jira), OpenWrt core (Bugzilla) keep a product tracker. HA and OPNsense let GitHub Issues follow the repo and add an architecture/plugins pointer. FWOS already chose “all product issues stay on `fwos`,” which matches the **hub tracker** family (VyOS/IPFire/TrueNAS OS) more than HA's per-repo issues.

6. **Runtime composition needs a pin file if image and apps ship separately.** HA `version/stable.json` is the explicit contract (Supervisor digest, Core per machine, OS per board, helper images). ublue relies on image tags + Renovate digest pins inside each product Containerfile. OPNsense/IPFire/OpenWrt pin via the package set built into that release. If FWOS host image and container apps are different remotes, something like HA `version` or ublue digest pins is required; it can live in the image repo or the hub, but it has to exist.

7. **Vendor daemons as OCI, not forks.** HA add-ons wrap Mosquitto/MariaDB; TrueNAS apps wrap upstream images; fedora-bootc examples Quadlet-embed registry images; OPNsense plugins wrap ports (FRR, Unbound, …). None of those projects git-vendor Kea/FRR/Unbound into the appliance org except as packaging.

---

## Source index

- OPNsense architecture: <https://docs.opnsense.org/development/architecture.html>
- OPNsense workflow: <https://docs.opnsense.org/development/workflow.html>
- OPNsense tools: <https://github.com/opnsense/tools/blob/master/README.md>
- OPNsense core: <https://github.com/opnsense/core/blob/master/README.md>, <https://github.com/opnsense/core/blob/master/CONTRIBUTING.md>
- OPNsense plugins: <https://github.com/opnsense/plugins/blob/master/README.md>
- OPNsense firmware UI: <https://docs.opnsense.org/manual/firmware.html>
- IPFire sources: <https://www.ipfire.org/docs/devel/sources>
- IPFire git guide: <https://www.ipfire.org/docs/devel/git>
- IPFire 2.x build: <https://www.ipfire.org/docs/devel/ipfire-2-x/build-howto>
- IPFire add-ons: <https://www.ipfire.org/docs/devel/ipfire-2-x/addon-howto>
- IPFire Bugzilla policy: <https://www.ipfire.org/docs/devel/bugzilla>
- IPFire 2.x tree (GitHub mirror): <https://github.com/ipfire/ipfire-2.x>
- HA architecture: <https://developers.home-assistant.io/docs/architecture_index>
- HA OS: <https://developers.home-assistant.io/docs/operating-system>, <https://github.com/home-assistant/operating-system/blob/dev/README.md>, <https://developers.home-assistant.io/docs/operating-system/getting-started>
- HA Supervisor: <https://developers.home-assistant.io/docs/supervisor>, <https://github.com/home-assistant/supervisor/blob/main/README.md>
- HA Apps: <https://developers.home-assistant.io/docs/apps>, <https://developers.home-assistant.io/docs/apps/repository>, <https://developers.home-assistant.io/docs/apps/publishing>, <https://github.com/home-assistant/addons/blob/master/README.md>
- HA version pins: <https://github.com/home-assistant/version/blob/master/stable.json>
- HA architecture RFC repo: <https://github.com/home-assistant/architecture/blob/master/README.md>
- bootc: <https://github.com/bootc-dev/bootc/blob/main/README.md>, <https://bootc-dev.github.io/bootc/bootc-images.html>
- Fedora bootc base-images: <https://gitlab.com/fedora/bootc/base-images/-/blob/main/README.md>
- Fedora bootc examples: <https://gitlab.com/fedora/bootc/examples/-/blob/main/README.md>, <https://gitlab.com/fedora/bootc/examples/-/blob/main/embed-workloads/README.md>
- Fedora CoreOS config: <https://github.com/coreos/fedora-coreos-config/blob/testing-devel/README.md>
- ublue image-template: <https://github.com/ublue-os/image-template/blob/main/README.md>
- ublue main: <https://github.com/ublue-os/main/blob/main/README.md>
- ublue packages: <https://github.com/ublue-os/packages/blob/main/README.md>
- ublue ucore: <https://github.com/ublue-os/ucore/blob/main/README.md>
- Bluefin contributing / split: <https://docs.projectbluefin.io/contributing>
- fedora-sysexts: <https://github.com/fedora-sysexts/fedora/blob/main/README.md>
- OpenWrt README + feeds: <https://github.com/openwrt/openwrt/blob/master/README.md>, <https://github.com/openwrt/openwrt/blob/master/feeds.conf.default>
- OpenWrt packages contributing: <https://github.com/openwrt/packages/blob/master/CONTRIBUTING.md>
- Turris repos: <https://docs.turris.cz/geek/contributing/repositories/>
- VyOS build: <https://github.com/vyos/vyos-build/blob/rolling/README.md>
- VyOS 1x consolidation: <https://github.com/vyos/vyos-1x/blob/rolling/README.md>
- VyOS development / Phabricator: <https://docs.vyos.io/en/latest/contributing/development.html>
- TrueNAS middleware: <https://github.com/truenas/middleware/blob/master/README.md>
- TrueNAS scale-build: <https://github.com/truenas/scale-build/blob/master/README.md>
- TrueNAS apps: <https://github.com/truenas/apps/blob/master/README.md>, <https://github.com/truenas/apps/blob/master/CONTRIBUTIONS.md>
