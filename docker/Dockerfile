# Matthew R. Wilson <mwilson@mattwilson.org>
# https://github.com/racingmars/ScarletDME/

### Build stage w/ gcc ########################################################
FROM debian:stable AS build

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install build-essential libpam-dev

COPY gplsrc /tmp/scarletdme/gplsrc/
COPY qmsys /tmp/scarletdme/qmsys/
COPY gpl.src Makefile pcode terminfo.src /tmp/scarletdme/
WORKDIR /tmp/scarletdme
RUN make

### Final container ###########################################################
FROM debian:stable

LABEL org.opencontainers.image.authors="Matthew R. Wilson <mwilson@mattwilson.org>"
LABEL org.opencontainers.image.url="https://github.com/racingmars/ScarletDME"
LABEL org.opencontainers.image.title="racingmars' ScarletDME build"

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install --no-install-recommends \
        nvi \
        openssh-server \
        supervisor \
        xinetd \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /qmdata

# sshd setup: each instance will generate its own host keys (in the data
# volume by initqmenv); we don't want to ship pre-made SSH host keys inside
# the container.
RUN rm -f /etc/ssh/*_key.* \
    && mkdir -p /var/run/sshd && chmod 0755 /var/run/sshd \
    && echo "HostKey /qmdata/ssh/ssh_host_ed25519_key" >> \
        /etc/ssh/sshd_config.d/container.conf \
    && echo "AuthorizedKeysFile /qmdata/ssh/keys-%u/authorized_keys" >> \
        /etc/ssh/sshd_config.d/container.conf \
    && echo "AllowTcpForwarding no" >> \
        /etc/ssh/sshd_config.d/container.conf \
    && echo "X11Forwarding no" >> \
        /etc/ssh/sshd_config.d/container.conf

RUN groupadd -g 42000 qmusers && adduser root qmusers

COPY --from=build /tmp/scarletdme/qmsys /usr/qmsys/
COPY --from=build /tmp/scarletdme/bin/* /usr/qmsys/bin/
COPY docker/support_files/scarlet.conf \
     docker/support_files/supervisord.conf \
        /etc/
COPY docker/support_files/qmserver \
     docker/support_files/qmclient \
        /etc/xinetd.d/
COPY docker/support_files/startqm \
     docker/support_files/initqmenv \
     docker/support_files/initssh \
        /usr/local/bin/

RUN echo "qmserver 4242/tcp # QMServer" >> /etc/services
RUN echo "qmclient 4243/tcp # QMClient" >> /etc/services

RUN chown -R root:qmusers /qmdata /usr/qmsys \
    && chmod -R g+rwX /qmdata /usr/qmsys

VOLUME /qmdata
EXPOSE 22 4242 4243

CMD ["/usr/local/bin/startqm"]
