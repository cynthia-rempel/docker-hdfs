# Creates pseudo distributed hadoop

FROM docker.io/library/centos:centos7.7.1908

# Put this above yum install until we know (for sure) which packages we want
RUN curl -s http://apache.cs.utah.edu/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz | tar -xz -C /usr/local/ && \
  mv /usr/local/hadoop-3.1.3 /usr/local/hadoop

RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
  yum -y install \
    curl \
    which \
    tar \
    sudo \
    openssh-server \
    openssh-clients \
    rsync \
    libselinux \
    java-1.8.0-openjdk

# passwordless ssh
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key && \
  ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key && \
  ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa && \
  cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

ENV HADOOP_PREFIX /usr/local/hadoop 
ENV HADOOP_COMMON_HOME /usr/local/hadoop 
ENV HADOOP_HDFS_HOME /usr/local/hadoop 
ENV HADOOP_MAPRED_HOME /usr/local/hadoop 
ENV HADOOP_YARN_HOME /usr/local/hadoop 
ENV HADOOP_CONF_DIR /usr/local/hadoop/etc/hadoop 
ENV YARN_CONF_DIR $HADOOP_PREFIX/etc/hadoop 
ENV HADOOP_HOME=/usr/local/hadoop 
ENV JAVA_HOME=/usr

RUN mkdir -p /usr/local/hadoop/input && \
  cp /usr/local/hadoop/etc/hadoop/*.xml /usr/local/hadoop/input

# pseudo distributed
COPY files/usr/local/hadoop/etc/hadoop/* /usr/local/hadoop/etc/hadoop/
RUN sed 's/HOSTNAME/localhost/' /usr/local/hadoop/etc/hadoop/core-site.xml.template > /usr/local/hadoop/etc/hadoop/core-site.xml

RUN /usr/local/hadoop/bin/hdfs namenode -format

RUN chmod 600 /root/.ssh/config
RUN chown root:root /root/.ssh/config

ADD bootstrap.sh /etc/bootstrap.sh
RUN chown root:root /etc/bootstrap.sh
RUN chmod 700 /etc/bootstrap.sh

ENV BOOTSTRAP /etc/bootstrap.sh

# fix the 254 error code
RUN sed  -i "/^[^#]*UsePAM/ s/.*/#&/"  /etc/ssh/sshd_config
RUN echo "UsePAM no" >> /etc/ssh/sshd_config
RUN echo "Port 2122" >> /etc/ssh/sshd_config

RUN service sshd start && $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh && $HADOOP_PREFIX/sbin/start-dfs.sh && $HADOOP_PREFIX/bin/hdfs dfs -mkdir -p /user/root
RUN service sshd start && $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh && $HADOOP_PREFIX/sbin/start-dfs.sh && $HADOOP_PREFIX/bin/hdfs dfs -put $HADOOP_PREFIX/etc/hadoop/ input

CMD ["/etc/bootstrap.sh", "-d"]

# Hdfs ports
EXPOSE 50010 50020 50070 50075 50090 8020 9000
# Mapred ports
EXPOSE 10020 19888
#Yarn ports
EXPOSE 8030 8031 8032 8033 8040 8042 8088
#Other ports
EXPOSE 49707 2122
