FROM rhel7
MAINTAINER "Tolik Litovsky" <tlitovsk@redhat.com>
ENV container docker
RUN yum -y update && yum clean all
RUN yum -y install yum-utils
RUN yum-config-manager --enable rhel-7-server-extras-rpms
RUN yum-config-manager --enable rhel-7-server-optional-rpms
RUN yum -y localinstall http://resources.ovirt.org/pub/yum-repo/ovirt-release35.rpm
RUN yum -y install vdsm vdsm-cli net-tools && yum clean all;
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done);
RUN rm -f /lib/systemd/system/multi-user.target.wants/*;
RUN rm -f /etc/systemd/system/*.wants/*;
RUN rm -f /lib/systemd/system/local-fs.target.wants/*;
RUN rm -f /lib/systemd/system/sockets.target.wants/*udev*;
RUN rm -f /lib/systemd/system/sockets.target.wants/*initctl*;
RUN rm -f /lib/systemd/system/basic.target.wants/*;
RUN rm -f /lib/systemd/system/anaconda.target.wants/*;

#fix language
RUN localedef -i en_US -c -f UTF-8 en_US.UTF-8

#prepeare the libvirtd
RUN echo "listen_tls = 0" >> /etc/libvirt/libvirtd.conf; \
echo 'listen_tcp = 1' >> /etc/libvirt/libvirtd.conf; \
echo 'tls_port = "16514"' >> /etc/libvirt/libvirtd.conf; \
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf; \
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf

RUN echo 'LIBVIRTD_ARGS="--listen"' >> /etc/sysconfig/libvirtd
RUN mkdir -p /var/lib/libvirt/images/

# Edit the service file which includes ExecStartPost to chmod /dev/kvm
RUN sed -i "/Service/a ExecStartPost=\/bin\/chmod 666 /dev/kvm" /usr/lib/systemd/system/libvirtd.service
EXPOSE 16509
RUN systemctl enable virtlockd
RUN systemctl enable libvirtd

#kill the modprobe , you cant from container
RUN sed -i /modprobe/d /usr/lib/systemd/system/multipathd.service

#perp the vdsm
RUN sed -i 's/# ssl = true/ssl = false/' /etc/vdsm/vdsm.conf
RUN sed -i 's/# management_port = 54321/management_port = 54321/' /etc/vdsm/vdsm.conf
RUN vdsm-tool configure --force
EXPOSE 54321
EXPOSE 4044
RUN systemctl enable vdsmd

VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
