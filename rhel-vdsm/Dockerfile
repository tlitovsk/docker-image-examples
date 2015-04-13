FROM rhel7
MAINTAINER "Tolik Litovsky" <tlitovsk@redhat.com>
ENV container docker
RUN yum -y update && yum clean all
RUN yum -y install yum-utils
RUN yum-config-manager --enable rhel-7-server-extras-rpms
RUN yum-config-manager --enable rhel-7-server-optional-rpms
RUN yum-config-manager --enable rhel-7-server-rhev-mgmt-agent-rpms
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done);
RUN rm -f /lib/systemd/system/multi-user.target.wants/*;
RUN rm -f /etc/systemd/system/*.wants/*;
RUN rm -f /lib/systemd/system/local-fs.target.wants/*;
RUN rm -f /lib/systemd/system/sockets.target.wants/*udev*;
RUN rm -f /lib/systemd/system/sockets.target.wants/*initctl*;
RUN rm -f /lib/systemd/system/basic.target.wants/*;
RUN rm -f /lib/systemd/system/anaconda.target.wants/*;

#start libvirt install
RUN yum -y install libvirt-daemon-driver-* libvirt-daemon libvirt-daemon-kvm qemu-kvm && yum clean all;
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
RUN systemctl enable libvirtd
#libvirt installed

#install kdump and tools
RUN yum -y install kexec-tools && yum clean all;
#done

#Start vdsm install
RUN yum -y install vdsm && yum clean all;
#kill the modprobe , you cant from container
RUN sed -i /modprobe/d /usr/lib/systemd/system/multipathd.service

#perp the vdsm
RUN sed -i 's/# ssl = true/ssl = false/' /etc/vdsm/vdsm.conf
RUN sed -i 's/# management_port = 54321/management_port = 54321/' /etc/vdsm/vdsm.conf
RUN vdsm-tool configure --force
EXPOSE 54321
EXPOSE 4044
RUN systemctl enable vdsmd
#vdsm installed

#debug extras
RUN yum -y install vdsm-cli net-tools && yum clean all;

VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
