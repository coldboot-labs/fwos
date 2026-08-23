# How the host image consumes artifacts from other repositories

Ticket: [coldboot-labs/fwos#3](https://github.com/coldboot-labs/fwos/issues/3)

**Question.** Given a Fedora **bootc** Containerfile in one git repository, what are the supported ways to consume (a) Rust binaries and (b) OCI images produced in other git repositories, at image-build time?

This note records first-party facts. It does not pick a topology.

## Two clocks

bootc work has two different “build” moments. They consume different things.

1. **Containerfile / `podman build` / `buildah build`.** This is image-build time for the host OS. Fedora bootc is an ordinary OCI image derived from a base such as `quay.io/fedora/fedora-bootc`, built with the same Containerfile tools as application containers. ([Fedora bootc getting started](https://docs.fedoraproject.org/en-US/bootc/getting-started/); source: [gitlab.com/fedora/bootc/docs `getting-started.adoc`](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/getting-started.adoc); [bootc building guidance](https://github.com/bootc-dev/bootc/blob/main/docs/src/building/guidance.md); [containers/common Containerfile(5)](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))

2. **`bootc-image-builder` (now in `osbuild/image-builder`).** This converts an *already built* bootc container image into a disk artifact (qcow2, raw, ISO, AMI, …). It does not compile Rust, clone other git repos, or `COPY` files from a Containerfile context. Its positional argument is an image reference. ([osbuild/bootc-image-builder README](https://github.com/osbuild/bootc-image-builder/blob/main/README.md); [osbuild/image-builder `bootc-image-builder/README.md`](https://github.com/osbuild/image-builder/blob/main/bootc-image-builder/README.md); [Fedora bootc getting started, “Conversion to Disk Images”](https://docs.fedoraproject.org/en-US/bootc/getting-started/))

The rest of this note is about clock (1), except the local/dev-registry section, which also covers how (2) sees a locally built image.

## Answer in one page

The image git repo **does not have to vendor binaries**. bootc has no special artifact protocol beyond OCI/Containerfile. Other repositories publish either **files** (binaries, configs) or **OCI images**; the host Containerfile consumes them with ordinary container-build instructions.

| Want | First-class at Containerfile time | Do not treat as a long-lived pin |
| --- | --- | --- |
| (a) Rust binary into `/usr` | Multi-stage compile then `COPY --from=builder`; or `COPY --from=<other-image>` of a prebuilt binary image; or `COPY` a file that CI already dropped into the build context | GitHub Actions artifacts as a permanent store (90-day default retention); compiling in the final bootc stage |
| (b) OCI app/jail image | **Physically bound:** `skopeo copy` into the host image at build time. **Logically bound:** record a Quadlet + symlink under `/usr/lib/bootc/bound-images.d` (pull happens at `bootc install`/`upgrade`/`switch`, not packed in the host tar). **Floating:** Quadlet only; pull on demand after boot | `RUN podman pull …` inside the Containerfile (whiteouts / nested overlay) |

Sources for that table are cited in the sections below.

## (a) Rust binaries at image-build time

Fedora bootc and RHEL image mode both treat a derived host image as a normal Containerfile. The getting-started template is: `FROM` a bootc base, `RUN dnf install`, `COPY` unpackaged application, `COPY` configuration, `RUN` config scripts. ([Fedora bootc getting started](https://docs.fedoraproject.org/en-US/bootc/getting-started/); [RHEL 10 image mode ch. 2](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-testing-rhel-bootc-images))

Put executables under `/usr`. The deployed host mounts most of the tree read-only; `/usr` is the OS-owned, versioned location. ([bootc building guidance, “Handling read-only vs writable locations”](https://github.com/bootc-dev/bootc/blob/main/docs/src/building/guidance.md); [Fedora bootc getting started, filesystem](https://docs.fedoraproject.org/en-US/bootc/getting-started/))

### A1. Multi-stage: compile elsewhere, `COPY --from=builder`

Fedora bootc **recommends multi-stage builds** when compiling. A bootc image is “not always the best environment to, for instance, compile a project”; compile in a builder stage and copy artifacts into the final stage. ([Fedora bootc building-containers](https://docs.fedoraproject.org/en-US/bootc/building-containers/); source: [building-containers.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/building-containers.adoc))

RHEL image mode shows the same pattern with a Go builder (the shape is identical for Rust: builder image has the toolchain; final `FROM` is `rhel-bootc`; `COPY --from=builder` the binary). It states the point of multi-stage: the deployment image must include only the application and its runtime, not build tools. You can also `COPY --from=<image>` from a **separate** image (local name, registry tag, or ID), not only from a named stage. ([RHEL 10 image mode §2.6](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-testing-rhel-bootc-images#benefits-of-custom-bootable-images-with-multi-stage-builds))

Containerfile `COPY --from=name` changes `<src>` from the build context to a named previous stage. ([Containerfile(5)](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))

`podman build` resolves `[name]` in this order: (1) `--build-context [name]=…`, (2) `AS [name]` stage, (3) image `[name]`, local or remote. Additional contexts may be a local directory, an HTTP tarball URL, or a container image (`container-image://`, `docker://`). ([podman-build(1) `--build-context`](https://docs.podman.io/en/latest/markdown/podman-build.1.html))

**Topology.** The host-image repo needs the Containerfile and a pin (digest/tag of the builder image, or source). It does not need the Rust source if the other repo already produced a binary image. If it compiles itself, it needs source in the build context (submodule, extra context, `RUN git clone`, or `cargo install --git`) and a builder stage that is **not** the final bootc image.

### A2. Fedora “artifact pattern”: `COPY --from=` another OCI image’s `/usr`

Fedora bootc documents an explicit **artifact pattern**: a custom bootc image references other container images as reusable components and copies configuration from them *before* further customizations:

```dockerfile
FROM quay.io/exampleos/baseconfig@sha256:.... as baseconfig
FROM scratch
COPY --from=builder /target-rootfs/ /
COPY --from=baseconfig /usr/ /usr/
```

([Fedora bootc building-containers, “Referencing configuration as a container artifact”](https://docs.fedoraproject.org/en-US/bootc/building-containers/); [building-containers.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/building-containers.adoc))

The same instruction copies a single binary: `COPY --from=ghcr.io/org/fwos-netd@sha256:… /usr/bin/netd /usr/bin/netd`. That is the Containerfile-native way to consume a Rust binary **produced in another git repo** without vendoring it.

Pin with `@sha256:…`. Fedora’s embedding docs: a tag may move on the registry; a digest names exactly one image. ([Fedora bootc embedding-containers, “Tagging, versioning and referencing images”](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [embedding-containers.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/embedding-containers.adoc)). GitHub Container Registry documents pull-by-digest for the same reason. ([GHCR, “Pull by digest”](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#pull-by-digest))

**Topology.** Other repos push OCI to a registry (including `ghcr.io`). The host-image repo stores only a digest (or tag) in the Containerfile. No binary blob in git.

### A3. `COPY` from the build context (CI artifacts, extra checkouts, `--build-context`)

`COPY <src> <dest>` copies from the **build context** (the directory passed to `podman build`, plus extra named contexts). `<src>` is relative to that context. Archives are **not** unpacked. (`ADD` may unpack local archives and may take a remote URL.) ([Containerfile(5) COPY / ADD](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))

That is how “COPY from CI artifacts” actually works: **CI, not Containerfile, fetches the file**; then `COPY` it. There is no Containerfile transport named “GitHub Actions artifact”.

`podman build --build-context name=../other-src` (or `https://…/src.tar`, or `container-image://…`) lets the Containerfile `COPY --from=name` without stuffing those files into the host-image git tree. Remote Podman does not support a local-directory extra context. ([podman-build(1) `--build-context`](https://docs.podman.io/en/latest/markdown/podman-build.1.html))

`RUN --mount=type=bind,from=…` can use the same named context without committing the files into a layer until you copy them. Bind `from` defaults to the build context; `from` may be a stage or image name. ([Containerfile(5) RUN mounts](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))

### A4. GitHub Actions artifacts (cross-repo)

GitHub Actions artifacts are files produced by a workflow run, uploaded with `actions/upload-artifact` and downloaded with `actions/download-artifact`. They exist to share files between jobs and to persist output after a run. ([GitHub: workflow artifacts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts); [storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts))

Cross-repo / cross-run download is supported only by elevating auth: `github-token` (token with `actions:read` on the target repo), `repository`, and `run-id`. Default permission is the current repo and current run. ([actions/download-artifact README, “Download Artifacts from other Workflow Runs or Repositories”](https://github.com/actions/download-artifact))

`gh run download RUN_ID` / `gh run download RUN_ID -n NAME` is the CLI equivalent. ([Downloading workflow artifacts](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/download-workflow-artifacts))

**Retention.** Default 90 days, then automatic deletion. Public repos: 1–90 days. Private: 1–400. Deleting a workflow run deletes its artifacts. ([Artifact and log retention](https://docs.github.com/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization); [Downloading workflow artifacts](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/download-workflow-artifacts); [Artifacts from deleted workflow runs](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts#artifacts-from-deleted-workflow-runs))

**Permissions in the zip.** Upload zipping strips mode: directories `755`, files `644`. An executable bit is not guaranteed after download unless the producer tars first and uploads that tar (`archive: false` on upload-artifact v7+). ([actions/download-artifact, “Maintaining File Permissions”](https://github.com/actions/download-artifact))

**Topology.** Usable as a **CI glue** into the host-image build context, not as the pin that lives in the Containerfile. A rebuild months later cannot rely on the artifact still existing. Does not force vendoring binaries in git; it does force the host-image **CI** to know `repository` + `run-id` (or to live in the same workflow).

### A5. GitHub Releases

A release asset has a stable download URL. Latest manually uploaded asset: `https://github.com/<owner>/<repo>/releases/latest/download/<asset-name>`. Older releases: the release’s own URL / tag. ([Linking to releases](https://docs.github.com/en/repositories/releasing-projects-on-github/linking-to-releases))

`gh release download [<tag>] -R owner/repo` downloads assets; without a tag it uses the latest release and requires `--pattern` or `--archive`. ([gh release download](https://cli.github.com/manual/gh_release_download))

At Containerfile time there are two first-party mechanisms:

- **`ADD <url> <dest>`** copies a remote file URL into the image. Local compressed archives unpack; URL download and unpack **cannot** be combined. ([Containerfile(5) ADD](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))
- **`RUN` + HTTP client.** Containerfile(5) documents `ARG TARGETARCH` specifically so a `RUN curl` can fetch an arch-specific binary. ([Containerfile(5) Platform/OS/Arch ARG](https://github.com/containers/common/blob/main/docs/Containerfile.5.md))

CI may also `gh release download` into the context and `COPY` (A3).

**Topology.** The host-image repo stores a URL (and should pin a **tag**, not only `latest`, if reproducibility matters). No binary in git. Relies on GitHub remaining reachable at build time unless CI vendor-copies the asset into the context. `latest` moves.

### A6. Git submodules

A submodule is a git repository mounted inside another. The superproject records a **gitlink** (the expected commit) plus `.gitmodules` (`submodule.<name>.url` / `.path`). Clone does **not** check out submodules unless you recurse (`git clone --recurse`, `git submodule update`, or `submodule.recurse`). ([gitsubmodules(7)](https://git-scm.com/docs/gitsubmodules))

GitHub Actions: `actions/checkout` `submodules: true` or `recursive`. Default is **false**. Checkout of a *different* repo (side-by-side or nested, without a submodule) is a separate `actions/checkout` with `repository:` and `path:`; private cross-repo needs a PAT, because `${{ github.token }}` is scoped to the current repository. ([actions/checkout README](https://github.com/actions/checkout/blob/main/README.md))

Once the other tree is in the build context, A3 `COPY` / multi-stage compile applies.

**Topology.** Submodules vendor a **source pin** (commit), not a binary. The host-image git repo grows a gitlink and `.gitmodules`; it does not have to store binaries. CI must recurse. Independent history of the child repo is preserved; the host-image repo must bump the gitlink to pick up new commits.

### A7. `cargo install` from a git URL

Cargo supports installing a binary crate from git:

```
cargo install --git <url> [--branch|--tag|--rev] [--locked] [--root <dir>] [crate]
```

Default source is crates.io; `--git`, `--path`, and `--registry` change it. Git installs may take `--branch`, `--tag`, or `--rev`. Default **ignores** `Cargo.lock` unless `--locked`. Default install root is `--root`, else `CARGO_INSTALL_ROOT`, else `install.root`, else `CARGO_HOME`, else `$HOME/.cargo` — **not** `/usr`. `--root` selects the installation root (`bin/` under it). `--path` always rebuilds. (`CARGO_TARGET_DIR` avoids a throwaway target dir, useful in CI.) ([cargo-install(1)](https://doc.rust-lang.org/cargo/commands/cargo-install.html))

Combined with Fedora’s multi-stage recommendation (A1): run `cargo install --git … --locked --rev <sha> --root /out` in a **builder** stage that has `rustc`, then `COPY --from=builder /out/bin/foo /usr/bin/foo` into the bootc stage. Doing `cargo install` in the final bootc stage would pull a compiler toolchain into the OS image unless a later layer deletes it; Fedora/RHEL multi-stage text exists to avoid that. ([Fedora bootc building-containers](https://docs.fedoraproject.org/en-US/bootc/building-containers/); [RHEL 10 image mode §2.6](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-testing-rhel-bootc-images#benefits-of-custom-bootable-images-with-multi-stage-builds))

**Topology.** Host-image repo stores a git URL + rev. Compiles at every host-image build (needs network to the git host and crates.io unless vendored/`--offline`). No binary in git. Reproducibility requires `--locked` and `--rev`/`--tag`, not a floating branch.

### A8. RPMs via `dnf` (only if the other repo actually ships RPMs)

`RUN dnf install` in a derived bootc image is the same as in an application container. Fedora bootc includes `dnf` in the default image. ([Fedora bootc dnf](https://docs.fedoraproject.org/en-US/bootc/dnf/); [dnf.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/dnf.adoc); [bootc building guidance, “Installing software”](https://github.com/bootc-dev/bootc/blob/main/docs/src/building/guidance.md))

That is a supported consumption path **if** the other repository publishes rpm-md. It is not how a raw `cargo build` artifact is consumed.

Do **not** `dnf -y update` as a general derived-image practice. ([Fedora bootc building-containers](https://docs.fedoraproject.org/en-US/bootc/building-containers/))

## (b) OCI images at image-build time

Fedora bootc lists three ways to get workload images onto a bootc host, plus Quadlets as the usual declaration format. ([Fedora bootc embedding-containers](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [embedding-containers.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/embedding-containers.adoc); [running-containers](https://docs.fedoraproject.org/en-US/bootc/running-containers/))

### B0. Do not `RUN podman pull` in the Containerfile

OCI layers use whiteouts (`.wh` / overlay `0:0` char devices). Nested `RUN podman pull quay.io/…` writes those whiteouts into the image filesystem and “will create problems”. Special runtime work is required; it is not the supported embedding path. ([bootc building guidance, “Nesting OCI containers in bootc containers”](https://github.com/bootc-dev/bootc/blob/main/docs/src/building/guidance.md); tracker [bootc-dev/bootc#128](https://github.com/bootc-dev/bootc/issues/128))

### B1. Quadlet files copied at build time (always)

Quadlets are files. They can live next to the Containerfile and `COPY` into `/etc/containers/systemd` or `/usr/share/containers/systemd`. That is image-build-time consumption of **unit text**, not of the referenced OCI bytes. ([Fedora bootc embedding-containers](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [running-containers](https://docs.fedoraproject.org/en-US/bootc/running-containers/); [podman-systemd.unit(5)](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html))

RHEL image mode repeats the same Quadlet-in-Containerfile example on `fedora-bootc`. ([How to embed containers on image mode for RHEL](https://developers.redhat.com/articles/2025/05/29/how-embed-containers-image-mode-rhel) — Red Hat Developers first-party article)

### B2. On-demand / floating images (not packed at host-image build)

If the Quadlet’s `Image=` is not present locally, Podman pulls when the unit starts. Easy; delays boot; fails disconnected. Application images remain physically distinct from the bootc image and are fetched over the network. ([Fedora bootc embedding-containers, “Default: on-demand pulls”](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [running-containers, “Images are still separate”](https://docs.fedoraproject.org/en-US/bootc/running-containers/))

Lifecycle can still be *logically* bound to the host by pinning `Image=` to a version tag or digest inside the host image; updating the host image then updates the pin. `podman-auto-update` is the opposite: the workload floats. ([running-containers](https://docs.fedoraproject.org/en-US/bootc/running-containers/))

**Topology.** Host-image repo stores Quadlet files and an image reference. Other repo publishes to a registry. Host image tarball does **not** contain the app layers.

### B3. Logically bound images (reference at build; pull at install/upgrade)

bootc pulls listed images during `bootc install`, `bootc upgrade`, and `bootc switch` into bootc-owned storage (`/usr/lib/bootc/storage`). Declare by symlink from `/usr/lib/bootc/bound-images.d/` to a Quadlet `.image` or `.container`. Podman uses `additionalimagestore=/usr/lib/bootc/storage` (per-container `GlobalArgs=`, **not** a global `storage.conf` extra store). ([bootc logically-bound-images](https://github.com/bootc-dev/bootc/blob/main/docs/src/logically-bound-images.md); [Fedora bootc embedding-containers](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [RHEL 10 image mode ch. 3](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-managing-logically-bound-images))

Properties from those docs:

- Host image can update without re-downloading unchanged app layers; app image can change without rebuilding the host (useful in development).
- Images are present at boot (unlike a first-boot `podman pull`).
- `bootc install` requires them already in the default `/var/lib/containers` store; they are copied to the target.
- Anaconda does **not** currently support logically bound images.
- Only the Quadlet `Image=` field is honored; `PullSecret=` in the unit is not.
- Default pull secret is `/etc/ostree/auth.json`.
- Garbage collection is bootc’s, tied to `bound-images.d`.
- Offline/disconnected install must **mirror every bound image**, not just the host image.

**Topology.** Host-image repo vendors Quadlet + symlink, not image layers. Other git repos publish OCI. Build-time of the *host Containerfile* does not download those layers; *install/upgrade* does.

### B4. Physically bound images (layers copied at host-image build)

For a fully self-contained host image (offline / no registry at runtime), copy OCI into the host filesystem **during `podman build`**, then copy into Podman’s store at first boot.

Documented instruction:

```
RUN skopeo copy --preserve-digests docker://<IMAGE> dir:/usr/lib/containers-image-cache/<DIRECTORY>
```

Runtime:

```
skopeo copy --preserve-digests dir:/usr/lib/containers-image-cache/<DIRECTORY> containers-storage:<IMAGE>
```

Do not use an additional image store for this: physically bound content changes with each OS update and that store is not designed to be swapped. The `dir:` transport preserves digest; you must also keep the image name. Fedora ships example scripts `embed_image.sh` / `copy_embedded_images.sh`. The copy-to-mutable-store step must run **before** any Quadlet that needs the image. ([Fedora bootc embedding-containers](https://docs.fedoraproject.org/en-US/bootc/embedding-containers/); [fedora/bootc/examples physically-bound-images](https://gitlab.com/fedora/bootc/examples/-/tree/main/physically-bound-images); [RHEL 10 image mode ch. 4](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-managing-physically-bound-images))

RHEL: physically bound images “contain the data in the final tar of the image, while logically bound images only reference the image and pull it during a bootc upgrade.” Trade-off: offline reliability vs rebuilding the entire base image for a small app change. ([RHEL 10 image mode ch. 4](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-managing-physically-bound-images))

`dir:`, `docker://`, `containers-storage:`, `oci:`, `oci-archive:` are the containers/image transports used by skopeo/podman/buildah/bootc. ([containers-transports(5)](https://github.com/containers/image/blob/main/docs/containers-transports.5.md))

**Topology.** Host-image **build** pulls other repos’ OCI and vendors the **layers inside the host image**. The git repo still need not vendor those layers; the registry does. Host image size and update payload grow with every app change.

## Local / dev registry and local storage

### Insecure local registry

bootc uses containers/image (same family as podman). It has **no** `--tls-verify=false`. Disable TLS per registry in `/etc/containers/registries.conf.d`, e.g. `location="localhost:5000"` `insecure=true`. Private registries use `/etc/ostree/auth.json`. Mirrors use registries.conf remapping. ([bootc registries-and-offline](https://github.com/bootc-dev/bootc/blob/main/docs/src/registries-and-offline.md); [containers-registries.conf(5)](https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md))

skopeo example of registry-to-local-registry: `skopeo copy docker://docker.io/library/alpine:latest docker://localhost:5000/alpine:latest`. ([containers-transports(5) examples](https://github.com/containers/image/blob/main/docs/containers-transports.5.md))

### `containers-storage` (podman’s local store)

To boot a locally built image without pushing: `bootc switch --transport containers-storage <name>`. Host bootc storage is **not** the same as podman storage; this command copies from podman into bootc. ([bootc booting-local-builds](https://github.com/bootc-dev/bootc/blob/main/docs/src/booting-local-builds.md); [containers-transports(5)](https://github.com/containers/image/blob/main/docs/containers-transports.5.md))

Offline USB-style updates: `skopeo copy docker://… oci:/path` or `dir:`; then `bootc switch --transport oci|dir …`. If the image is already local, `skopeo copy containers-storage:[image]:[tag] dir:…`. ([bootc registries-and-offline](https://github.com/bootc-dev/bootc/blob/main/docs/src/registries-and-offline.md); [Fedora bootc disconnected-updates](https://docs.fedoraproject.org/en-US/bootc/disconnected-updates/); [disconnected-updates.adoc](https://gitlab.com/fedora/bootc/docs/-/blob/main/modules/ROOT/pages/disconnected-updates.adoc))

### `bootc-image-builder` and local images

Every documented `podman run … bootc-image-builder` example bind-mounts host container storage:

`-v /var/lib/containers/storage:/var/lib/containers/storage`

(rootless experimental: `~/.local/share/containers/storage` plus `--in-vm`). That is how a locally built (or locally pulled) bootc image is visible to the converter. The tool’s input is still an image reference, not a git tree. ([bootc-image-builder README](https://github.com/osbuild/bootc-image-builder/blob/main/README.md); same text in [osbuild/image-builder](https://github.com/osbuild/image-builder/blob/main/bootc-image-builder/README.md))

The `osbuild/bootc-image-builder` GitHub repository is **archived**; code and issues moved to [`osbuild/image-builder`](https://github.com/osbuild/image-builder). ([bootc-image-builder README, “Migration”](https://github.com/osbuild/bootc-image-builder/blob/main/README.md))

`COPY --from=localhost/…` / `COPY --from=<local tag>` during `podman build` is the same mechanism as A2, using local storage instead of a remote registry. ([RHEL 10 image mode §2.6](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-testing-rhel-bootc-images#benefits-of-custom-bootable-images-with-multi-stage-builds); [podman-build(1) `--from` / `--build-context`](https://docs.podman.io/en/latest/markdown/podman-build.1.html))

## What bootc-image-builder / Fedora image mode add (and do not)

- **Image mode** = OCI-native OS: same Containerfile/Podman/CI techniques as app containers. ([RHEL 10 image mode ch. 1](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/introducing-image-mode-for-rhel); [Fedora bootc getting started](https://docs.fedoraproject.org/en-US/bootc/getting-started/))
- **bib** customizations (`config.toml` users, filesystems, kickstart) apply at **disk** conversion, not at Containerfile artifact consumption. Images may also embed `/usr/lib/bootc-image-builder/config.toml`. ([bootc-image-builder README, Build config](https://github.com/osbuild/bootc-image-builder/blob/main/README.md))
- **OCI artifacts** (the OCI artifacts spec / referrers API) are **not** what bootc uses. bootc-compatible images are ordinary OCI **images**. Encapsulating qcow2/AMI as an OCI artifact is out of bootc’s goals. ([bootc relationship-oci-artifacts](https://github.com/bootc-dev/bootc/blob/main/docs/src/relationship-oci-artifacts.md))
- Fedora GitHub Actions note: derived bootc images are large; runners may need extra disk; example uses Buildah/Podman Actions. That is CI hygiene, not an artifact-consumption API. ([Fedora bootc building-containers, GitHub Actions](https://docs.fedoraproject.org/en-US/bootc/building-containers/))

## Trade-offs that affect repo topology

Does the **image git repo** have to vendor binaries? **No.** Nothing in bootc, Fedora image mode, or bib requires binaries in that git tree. Supported pins are:

| Pin lives in | Binary/layers live in | Host-image git contains | Rebuild without the other repo’s CI? |
| --- | --- | --- | --- |
| Containerfile `COPY --from=image@sha256` | Registry (GHCR, Quay, local) | Digest + Containerfile | Yes, while the registry keeps the blob |
| Physically bound `skopeo copy` | Host image tar (after build); registry at build time | Image reference | Yes for already-built host image; no for rebuilding without registry/mirror |
| Logically bound Quadlet | Registry; pulled at install/upgrade | Quadlet + symlink | Host image rebuild yes; install needs registry or mirror of **all** bound images |
| Floating Quadlet | Registry at runtime | Quadlet | Host image independent of app bits |
| GitHub Release URL / `ADD` | GitHub Releases | URL + tag | Until the release asset is deleted |
| Actions artifact + `COPY` | GitHub Actions storage (90 days default) | Nothing durable | No, after expiry |
| Submodule + compile | Built in the host-image pipeline | gitlink + `.gitmodules` | Yes, if child commits stay fetchable |
| `cargo install --git --rev` | Built in the host-image pipeline | URL + rev | Yes, if git + crates stay fetchable |
| File committed in host-image repo | That git repo | The binary | Yes (true vendoring) |

Implications that matter for splitting `fwos` host-image vs daemon/UI/apps repos:

1. **Preferred split (binaries):** other repo builds a small OCI (or a bootc “artifact pattern” image) and pushes by digest; host Containerfile `COPY --from=` into `/usr/bin`. Host git stays a Containerfile + overlay. Matches Fedora’s artifact pattern and RHEL `COPY --from=<image>`.
2. **Preferred split (jails/apps):** other repo pushes OCI; host image either physically binds (appliance wants one artifact, offline) or logically binds (faster app iteration, extra mirrors on install). Fedora/RHEL document both; they disagree with `RUN podman pull`.
3. **Submodules / `cargo install --git`:** keep source out of the host-image tree as *blobs*, but the host-image **build** becomes a compile farm and takes a toolchain builder stage. Fedora explicitly prefers that compile not happen in the final bootc stage.
4. **Actions artifacts:** fine as a same-workflow handoff; a bad long-term contract between public repos (retention, zip modes, extra token + `run-id`).
5. **Releases:** better than Actions artifacts for a named version; still GitHub-HTTP at build time; pin tags.
6. **Vendoring binaries in the host-image git repo** is optional, not required, and is the only path that makes the host-image remote self-contained with **no** registry/GitHub/other-git at build time.
7. **bib** never sees the other git repos. Whatever you chose above must already be inside the bootc image (or, for logically bound, listed so `bootc install` / the installer environment can pull it). Anaconda does not support logically bound images today.

## Source index

| Document | URL |
| --- | --- |
| Fedora bootc getting started | https://docs.fedoraproject.org/en-US/bootc/getting-started/ |
| Fedora bootc building derived images | https://docs.fedoraproject.org/en-US/bootc/building-containers/ |
| Fedora bootc embedding containers | https://docs.fedoraproject.org/en-US/bootc/embedding-containers/ |
| Fedora bootc running containers | https://docs.fedoraproject.org/en-US/bootc/running-containers/ |
| Fedora bootc dnf | https://docs.fedoraproject.org/en-US/bootc/dnf/ |
| Fedora bootc disconnected updates | https://docs.fedoraproject.org/en-US/bootc/disconnected-updates/ |
| Fedora bootc docs source | https://gitlab.com/fedora/bootc/docs |
| Fedora bootc physically-bound example | https://gitlab.com/fedora/bootc/examples/-/tree/main/physically-bound-images |
| bootc building guidance | https://github.com/bootc-dev/bootc/blob/main/docs/src/building/guidance.md |
| bootc logically bound images | https://github.com/bootc-dev/bootc/blob/main/docs/src/logically-bound-images.md |
| bootc registries / offline | https://github.com/bootc-dev/bootc/blob/main/docs/src/registries-and-offline.md |
| bootc local builds | https://github.com/bootc-dev/bootc/blob/main/docs/src/booting-local-builds.md |
| bootc vs OCI artifacts | https://github.com/bootc-dev/bootc/blob/main/docs/src/relationship-oci-artifacts.md |
| Nested `podman pull` tracker | https://github.com/bootc-dev/bootc/issues/128 |
| RHEL 10 image mode: building | https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-testing-rhel-bootc-images |
| RHEL 10 image mode: logically bound | https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-managing-logically-bound-images |
| RHEL 10 image mode: physically bound | https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/building-and-managing-physically-bound-images |
| Containerfile(5) | https://github.com/containers/common/blob/main/docs/Containerfile.5.md |
| podman-build(1) | https://docs.podman.io/en/latest/markdown/podman-build.1.html |
| containers-transports(5) | https://github.com/containers/image/blob/main/docs/containers-transports.5.md |
| bootc-image-builder (archived README) | https://github.com/osbuild/bootc-image-builder/blob/main/README.md |
| image-builder (current bib) | https://github.com/osbuild/image-builder/blob/main/bootc-image-builder/README.md |
| cargo-install(1) | https://doc.rust-lang.org/cargo/commands/cargo-install.html |
| gitsubmodules(7) | https://git-scm.com/docs/gitsubmodules |
| GitHub Actions artifacts | https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts |
| download-artifact (cross-repo) | https://github.com/actions/download-artifact |
| Artifact retention | https://docs.github.com/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization |
| Linking to GitHub Releases | https://docs.github.com/en/repositories/releasing-projects-on-github/linking-to-releases |
| gh release download | https://cli.github.com/manual/gh_release_download |
| actions/checkout | https://github.com/actions/checkout/blob/main/README.md |
| GitHub Container Registry | https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry |
