FROM centos:7.4.1708

ARG RELEASE_VERSION="0.0.1"

# -----------------------------------------------------------------------------
# Set image metadata
# -----------------------------------------------------------------------------
LABEL maintainer "Samsung CNCT"

# -----------------------------------------------------------------------------
# - Import the RPM GPG keys for repositories
# - Base install of required packages
# -----------------------------------------------------------------------------
ADD clean-install /usr/local/bin/clean-install
RUN chmod +x /usr/local/bin/clean-install

RUN  rpm --import \
             http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7 \
     && rpm --import \
             https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7 \
     && rpm --import \
             https://dl.iuscommunity.org/pub/ius/IUS-COMMUNITY-GPG-KEY \
     && clean-install \
             centos-release-scl \
             centos-release-scl-rh \
             epel-release \
             https://centos7.iuscommunity.org/ius-release.rpm \
             openssh-clients-7.4p1-16.el7 \
             openssh-server-7.4p1-16.el7 \
             openssl-1.0.2k-16.el7 \
             sudo-1.8.23-3.el7 \
             yum-plugin-versionlock-1.1.31-50.el7 \
             less \
             systemd \
     && find /lib/systemd/system/sysinit.target.wants/ -name "systemd-tmpfiles-setup.service" -delete \
     && rm -f /lib/systemd/system/multi-user.target.wants/* \
              /etc/systemd/system/*.wants/* \
              /lib/systemd/system/local-fs.target.wants/* \
              /lib/systemd/system/sockets.target.wants/*udev* \
              /lib/systemd/system/sockets.target.wants/*initctl* \
              /lib/systemd/system/basic.target.wants/* \
     && echo "ReadKMsg=no" >> /etc/systemd/journald.conf

# -----------------------------------------------------------------------------
# Copy files into place
# -----------------------------------------------------------------------------
ADD src/usr/bin /usr/bin/
ADD src/usr/sbin /usr/sbin/
ADD src/etc /etc/

# Set up systemd volume to host cgroups
VOLUME ["/sys/fs/cgroup", "/var/run", "/var/lib/docker/containers"]

# -----------------------------------------------------------------------------
# Provisioning
# - UTC Timezone
# - Networking
# - Configure SSH defaults for non-root public key authentication
# - Enable the wheel sudoers group
# - Replace placeholders with values in systemd service unit template
# - Set permissions
# -----------------------------------------------------------------------------
RUN ln -sf \
                /usr/share/zoneinfo/UTC \
                /etc/localtime \
        && echo "NETWORKING=yes" \
                > /etc/sysconfig/network \
        && sed -i \
                -e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' \
                -e 's~^#PermitRootLogin yes~PermitRootLogin no~g' \
                -e 's~^#UseDNS yes~UseDNS no~g' \
                -e 's~^\(.*\)/usr/libexec/openssh/sftp-server$~\1internal-sftp~g' \
                /etc/ssh/sshd_config \
        && sed -i \
                -e 's~^# %wheel\tALL=(ALL)\tALL~%wheel\tALL=(ALL) ALL~g' \
                -e 's~\(.*\) requiretty$~#\1requiretty~' \
                /etc/sudoers \
        && sed -i \
                -e "s~{{RELEASE_VERSION}}~${RELEASE_VERSION}~g" \
                /etc/systemd/system/centos-ssh@.service \
        && chmod 644 \
                /etc/sshd-bootstrap.{conf,env} \
        && chmod 700 \
                /usr/{bin/healthcheck,sbin/sshd-{bootstrap,wrapper}} \
        && sed -ri 's#(^ExecStart=)#ExecStartPre=/usr/sbin/sshd-bootstrap --verbose\n\1#i' /usr/lib/systemd/system/sshd.service \
        && sed -ri 's#ExecStart=.*#ExecStart=/usr/sbin/sshd-wrapper#' /usr/lib/systemd/system/sshd.service \
        && sed -ri 's#^PermitRootLogin .*#PermitRootLogin yes#i' /etc/ssh/sshd_config \
        && systemctl enable sshd.service \
        && echo 'root:password' | chpasswd
        # SSH login fix. Otherwise user is kicked off after login
        #&& sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

EXPOSE 22

# -----------------------------------------------------------------------------
# Set default environment variables
# -----------------------------------------------------------------------------
ENV SSH_AUTHORIZED_KEYS="" \
        SSH_AUTOSTART_SSHD="true" \
        SSH_AUTOSTART_SSHD_BOOTSTRAP="true" \
        SSH_CHROOT_DIRECTORY="%h" \
        SSH_INHERIT_ENVIRONMENT="false" \
        SSH_PASSWORD_AUTHENTICATION="true" \
        SSH_SUDO="ALL=(ALL) ALL NOPASSWORD" \
        SSH_TIMEZONE="UTC" \
        SSH_USER="app-admin" \
        SSH_USER_FORCE_SFTP="false" \
        SSH_USER_HOME="/home/%u" \
        SSH_USER_ID="500:500" \
        SSH_USER_PASSWORD="" \
        SSH_USER_PASSWORD_HASHED="false" \
        SSH_USER_PRIVATE_KEY="" \
        SSH_USER_SHELL="/bin/bash"

HEALTHCHECK \
        --interval=1s \
        --timeout=1s \
        --retries=5 \
        CMD ["/usr/bin/healthcheck"]

# tell systemd that it is in docker (it will check for the container env)
# https://www.freedesktop.org/wiki/Software/systemd/ContainerInterface/
ENV container docker

# systemd exits on SIGRTMIN+3, not SIGTERM (which re-executes it)
# https://bugzilla.redhat.com/show_bug.cgi?id=1201657
STOPSIGNAL SIGRTMIN+3

# wrap systemd with our special entrypoint, see pkg/build for how this is built
# basically this just lets us set up some things before continuing on to systemd
# while preserving that systemd is PID1
# for how we leverage this, see pkg/cluster
COPY [ "entrypoint/entrypoint", "/usr/local/bin/" ]

# We need systemd to be PID1 to run the various services (docker, kubelet, etc.)
# NOTE: this is *only* for documentation, the entrypoint is overridden at runtime
#ENTRYPOINT [ "/usr/local/bin/entrypoint","/sbin/init","--log-target=journal" ]
ENTRYPOINT [ "/sbin/init","--log-target=journal" ]

# the docker graph must be a volume to avoid overlay on overlay
# NOTE: we do this last because changing a volume with a Dockerfile must
# occur before defining it.
# See: https://docs.docker.com/engine/reference/builder/#volume
VOLUME [ "/var/lib/docker" ]

CMD []
