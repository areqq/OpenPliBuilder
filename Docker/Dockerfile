# OpenPLi build host — Ubuntu 22.04 (the version OpenPLi recommends; 24.04+
# needs apparmor/coreutils workarounds, so we stay on 22.04 on purpose).
# The image is only the build *toolchain host*: you clone openpli-oe-core into
# the mounted /build volume and build there, so artifacts persist and the
# image stays reusable. See README for the run command and build steps.
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm-256color \
    TZ=Europe/Warsaw

WORKDIR /build

# Package groups (kept on one logical line; Dockerfile mid-RUN # comments are
# unreliable, so they are documented here instead):
#  1. OpenPLi host packages — wiki.openpli.org/Information_for_Developers (Ubuntu 22.04)
#  2. extra OE/Yocto (scarthgap) host deps that recipes commonly pull in
#  3. container quality-of-life tools
RUN apt-get update && apt-get -y upgrade && \
    apt-get install -y --no-install-recommends \
        sed wget cvs subversion git-core git-lfs \
        coreutils unzip texi2html texinfo docbook-utils \
        gawk python3-distutils diffstat help2man make gcc build-essential g++ \
        desktop-file-utils chrpath default-jre gettext zip libssl-dev lz4 \
        libproc-processtable-perl \
        rsync cpio file bzip2 xz-utils zstd python3 python3-pip python3-jinja2 \
        sudo ca-certificates locales curl nano vim mc ncurses-term procps iproute2 && \
    ( [ -e /usr/bin/makeinfo ] || ln -sf /usr/bin/texi2any /usr/bin/makeinfo ) && \
    echo "dash dash/sh boolean false" | debconf-set-selections && \
    dpkg-reconfigure -f noninteractive dash && \
    apt-get -y autoremove && apt-get -y clean && rm -rf /var/lib/apt/lists/* && \
    echo "root:root" | chpasswd && \
    useradd -m -s /bin/bash -u 1000 docker && echo "docker:docker" | chpasswd && \
    echo "docker ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/docker && \
    chown docker:docker /build

# UTF-8 locale (Yocto sanity check requires it)
RUN localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LC_ALL=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LANG=en_US.UTF-8

USER docker:docker

# git "dubious ownership" on the bind-mounted /build repos otherwise breaks
# bitbake's git fetches; also give a default identity for any commits.
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "docker@openplibuilder" && \
    git config --global user.name "OpenPLi Builder" && \
    git lfs install --skip-repo

CMD ["bash"]
