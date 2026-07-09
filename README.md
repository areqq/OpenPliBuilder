# OpenPliBuilder

Reproducible **OpenPLi build host** in a container — for building images and
individual packages (e.g. for **Vu+ Uno 4K SE**, `MACHINE=vuuno4kse`) against
the latest OpenPLi (Yocto *scarthgap*).

The image is just the toolchain host (Ubuntu 22.04 with all the packages
OpenPLi's [developer wiki](https://wiki.openpli.org/Information_for_Developers)
requires). You clone `openpli-oe-core` into a **mounted** `build/` directory
and build there, so the (large) build artifacts live on the host and survive
container restarts, while the image stays small and reusable.

> Ubuntu 22.04 on purpose — OpenPLi recommends it; 24.04+ needs apparmor /
> coreutils workarounds.

## 1. Build the image

```bash
git clone https://github.com/areqq/OpenPliBuilder
cd OpenPliBuilder
podman build -t areqq/openplibuilder:latest Docker      # or: docker build ...
```

## 2. Run the container

`build/` on the host is mounted as `/build` inside.

**podman (rootless):**
```bash
mkdir -p build
podman run -it --rm \
    --userns=keep-id:uid=1000,gid=1000 \
    -v ./build:/build:Z \
    areqq/openplibuilder:latest
```
`--userns=keep-id:uid=1000,gid=1000` maps your host user onto the container's
`docker` user (uid 1000, which owns `/build`), so files created by the build
are owned by *you* on the host and stay editable — no `root`-owned artifacts.

**docker:**
```bash
mkdir -p build
docker run -it --rm -v ./build:/build areqq/openplibuilder:latest
```

## 3. Get the sources (inside the container, in `/build`)

```bash
git clone https://github.com/OpenPLi/openpli-oe-core.git   # default = scarthgap (latest)
cd openpli-oe-core
make                 # 'init': generates env.source + conf, fetches bitbake / openembedded-core
make update          # sync layer submodules (run again later to update)
```
For a pinned stable release instead of bleeding-edge, clone a release branch:
`git clone -b release-9.2 https://github.com/OpenPLi/openpli-oe-core.git`.

`make` creates the actual build dir at **`openpli-oe-core/build/`** (that's
where `env.source`, `conf/` and all output live).

## 4a. Build the full image (Vu+ Uno 4K SE)

From the repo root (`openpli-oe-core`):
```bash
MACHINE=vuuno4kse make image
```
The flashable image + rootfs land under `openpli-oe-core/build/tmp/deploy/`.

## 4b. Build a single package

```bash
cd build
source env.source
MACHINE=vuuno4kse bitbake -k exteplayer3      # or any recipe: enigma2, gstplayer, ...
```
The built `.ipk` ends up in `openpli-oe-core/build/tmp/deploy/ipk/`.
Handy recipe commands: `bitbake -c cleanall <pkg>`, `bitbake -c devshell <pkg>`,
`bitbake -e <pkg>` (dump expanded env), `bitbake-layers show-recipes`.

## Notes / gotchas

- **Disk & time:** a first full build pulls tens of GB of sources and builds a
  cross-toolchain from scratch — expect **~100 GB** under `build/` and several
  hours. A single package still needs the toolchain + its deps built once, so
  the *first* `bitbake` is long even for a small recipe. Keep everything on
  **local block storage** (OpenPLi does not support building over NFS).
- **Never build as root** — the image already runs as the non-root `docker`
  user (bitbake refuses root). `sudo` is available inside if you need it.
- `/bin/sh` is bash (not dash) and the locale is `en_US.UTF-8`, both required
  by Yocto's sanity check; `git safe.directory=*` is set so bitbake's git
  fetches work on the bind-mounted, differently-owned repo.
- To rebuild just your recipe after editing it:
  `bitbake -c cleansstate <pkg> && bitbake <pkg>`.

## Links
- OpenPLi build docs: https://wiki.openpli.org/Information_for_Developers#Create_your_own_build
- Docker-image build thread: https://forums.openpli.org/topic/45586-building-openpli-using-a-docker-image/
- openpli-oe-core: https://github.com/OpenPLi/openpli-oe-core
