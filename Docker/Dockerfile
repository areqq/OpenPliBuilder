FROM ubuntu:22.04

ENV DEBIAN_FRONTEND noninteractive
ENV TERM xterm-256color
ENV TZ="Europe/Warsaw"

WORKDIR /build

RUN \ 
    apt-get update -y   && \
    apt-get upgrade -y  && \
    apt-get install -y --no-install-recommends \
        sed wget cvs subversion git-core \
        coreutils unzip texi2html texinfo docbook-utils sudo file \
        gawk python3-distutils diffstat help2man make gcc build-essential g++ libproc-processtable-perl \
        chrpath gettext zip libssl-dev cpio default-jre ca-certificates locales mc vim terminfo curl && \
    apt -y autoremove   && \
    apt -y autoclean    && \
    rm -Rf /var/cache/apt/archives/* && \
    echo "root:root" | chpasswd && \
    useradd -m docker && \
    echo "docker:docker" | chpasswd && \
    echo "docker ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/docker && \
    chown docker:docker /build

# default-jre locales desktop-file-utils 

# Change the default system shell to bash (so /bin/sh will actually use bash instead of dash)
#RUN git clone https://github.com/OpenPLi/openpli-oe-core.git
#RUN cd openpli-oe-core ; make

# Set environment
#RUN locale-gen en_US.UTF-8
RUN localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LC_ALL="en_US.UTF-8"
ENV LANGUAGE="en_US:en"
ENV LANG="en_US.UTF-8"

USER docker:docker

CMD ["bash"]
