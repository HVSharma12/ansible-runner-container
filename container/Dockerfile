#!BuildTag: ansible-runner:latest

FROM opensuse/tumbleweed:latest

ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

RUN mkdir -p /container
COPY label-install \
    label-uninstall \
    ansible-wrapper.sh \
    /container/

RUN chmod +x /container/ansible-wrapper.sh

WORKDIR /work

LABEL INSTALL="/usr/bin/podman run --env IMAGE=IMAGE --rm --security-opt label=disable -v /:/host IMAGE /bin/bash /container/label-install"
LABEL UNINSTALL="/usr/bin/podman run --rm --security-opt label=disable -v /:/host IMAGE /bin/bash /container/label-uninstall"
LABEL USER-INSTALL="/usr/bin/podman run --env IMAGE=IMAGE --security-opt label=disable --rm -v \${PWD}/:/host IMAGE /bin/bash /container/label-install"
LABEL USER-UNINSTALL="/usr/bin/podman run --rm --security-opt label=disable -v \${PWD}/:/host IMAGE /bin/bash /container/label-uninstall"

RUN zypper -v -n in \
    ansible \
    ansible-core \
    ansible-runner \
    git-core \
    openssh-clients \
    python3-libvirt-python \
    python3-lxml \
    python3-netaddr \
    vim \
    ; \
    zypper clean --all
