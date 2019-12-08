FROM centos/centos7.7.1908

COPY coudera.repo /etc/yum.repos.d/cloudera.repo

RUN dnf -y install hadoop-hdfs-fuse

RUN useradd user

COPY fstab /etc/fstab

USER user

ENV LIBHDFS_OPTS="-Xms64m -Xmx256m"

# References: https://docs.cloudera.com/documentation/enterprise/latest/topics/cdh_ig_hdfs_mountable.html
