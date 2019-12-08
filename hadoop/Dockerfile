# Creates pseudo distributed hadoop

FROM docker.io/library/centos:centos7.7.1908

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

# attempt to do use users and groups
RUN useradd -r hdfs 

RUN curl -s http://apache.cs.utah.edu/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz | tar -xz -C /usr/local/ --group=hdfs --owner=hdfs && \
  mv /usr/local/hadoop-3.1.3 /usr/local/hadoop 


# passwordless ssh
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key && \
  ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key && \
  ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa && \
  cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys && \
  mkdir -p /home/hdfs/.ssh && \
  cp /root/.ssh/id_rsa.pub /home/hdfs/.ssh/authorized_keys && \
  chown -R hdfs:hdfs /home/hdfs/.ssh

ENV HADOOP_COMMON_HOME=/usr/local/hadoop \
  HADOOP_HDFS_HOME=/usr/local/hadoop \
  HADOOP_MAPRED_HOME=/usr/local/hadoop \ 
  HADOOP_YARN_HOME=/usr/local/hadoop \
  HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop \
  YARN_CONF_DIR=/usr/local/hadoop/etc/hadoop \
  HADOOP_HOME=/usr/local/hadoop \
  HDFS_NAMENODE_USER=hdfs \
  HDFS_DATANODE_USER=hdfs \
  HDFS_SECONDARYNAMENODE_USER=hdfs

RUN mkdir -p /usr/local/hadoop/input && \
  cp /usr/local/hadoop/etc/hadoop/*.xml /usr/local/hadoop/input

# pseudo distributed
COPY files/usr/local/hadoop/etc/hadoop/* /usr/local/hadoop/etc/hadoop/
RUN sed 's/HOSTNAME/localhost/' /usr/local/hadoop/etc/hadoop/core-site.xml.template > /usr/local/hadoop/etc/hadoop/core-site.xml

RUN echo "export HADOOP_COMMON_HOME=/usr/local/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HADOOP_HDFS_HOME=/usr/local/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HADOOP_MAPRED_HOME=/usr/local/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HADOOP_YARN_HOME=/usr/local/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export YARN_CONF_DIR=/usr/local/hadoop/etc/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HADOOP_HOME=/usr/local/hadoop" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre/" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HDFS_NAMENODE_USER=hdfs" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HDFS_DATANODE_USER=hdfs" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
  echo "export HDFS_SECONDARYNAMENODE_USER=hdfs" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh


COPY files/etc/bootstrap.sh /etc/bootstrap.sh
RUN chown root:hdfs /etc/bootstrap.sh $HADOOP_HOME/etc/hadoop/hadoop-env.sh
RUN chmod 750 /etc/bootstrap.sh $HADOOP_HOME/etc/hadoop/hadoop-env.sh

RUN $HADOOP_HOME/etc/hadoop/hadoop-env.sh && \
  mkdir -p /usr/local/hadoop/logs && \
  chown -R hdfs:hdfs /usr/local/hadoop/logs && \
  su hdfs bash -c "usr/local/hadoop/bin/hdfs namenode -format"

ENV BOOTSTRAP /etc/bootstrap.sh

RUN systemctl enable sshd.service

RUN ls /usr/local/hadoop

RUN $HADOOP_HOME/etc/hadoop/hadoop-env.sh && \
  $HADOOP_HOME/sbin/start-dfs.sh && \
  $HADOOP_HOME/bin/hdfs dfs -mkdir -p /user/root
RUN $HADOOP_HOME/etc/hadoop/hadoop-env.sh && \
  $HADOOP_HOME/sbin/start-dfs.sh && \
  $HADOOP_HOME/bin/hdfs dfs -put $HADOOP_HOME/etc/hadoop/ input

CMD ["/etc/bootstrap.sh", "-d"]

# Hdfs ports
EXPOSE 50010 50020 50070 50075 50090 8020 9000
# Mapred ports
EXPOSE 10020 19888
#Yarn ports
EXPOSE 8030 8031 8032 8033 8040 8042 8088
#Other ports
EXPOSE 49707 2122
