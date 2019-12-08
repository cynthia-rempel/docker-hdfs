FROM centos/centos7.7.1908

# create our own repo file to enable (or disable) cloudera repos
COPY coudera.repo /etc/yum.repos.d/cloudera.repo

RUN cd /etc/pki/rpm-gpg/ && \
  wget https://archive.cloudera.com/cdh7/7.0.3.0/redhat7/yum/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins && \
  rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-Jenkins && \
  dnf -y install hadoop-hdfs-fuse

RUN useradd user

# TODO: add the needed lines
# COPY fstab /etc/fstab

USER user

ENV LIBHDFS_OPTS="-Xms64m -Xmx256m"

# References: https://docs.cloudera.com/documentation/enterprise/latest/topics/cdh_ig_hdfs_mountable.html
