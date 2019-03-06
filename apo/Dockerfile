ARG BASE_IMAGE="centos:centos7.4.1708"
FROM ${BASE_IMAGE}

# NOTE: ARCH must be defined again after FROM
# https://docs.docker.com/engine/reference/builder/#scope
ARG ARCH="amd64"

# https://hub.docker.com/_/centos/
# https://github.com/docker/docker/issues/7459
ENV container=docker

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;

STOPSIGNAL SIGRTMIN+3

RUN yum install -y openssh openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:password' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN ssh-keygen -A
RUN systemctl enable sshd.service

# We need systemd to be PID1 to run the various services (docker, kubelet, etc.)
# NOTE: this is *only* for documentation, the entrypoint is overridden at runtime
CMD [ "/sbin/init" ]

# the docker graph must be a volume to avoid overlay on overlay
# NOTE: we do this last because changing a volume with a Dockerfile must
# occur before defining it.
# See: https://docs.docker.com/engine/reference/builder/#volume
VOLUME [ "/var/lib/docker", "/run", "/tmp" ]
