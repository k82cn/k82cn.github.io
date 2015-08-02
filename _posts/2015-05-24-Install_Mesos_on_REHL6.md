---
layout: post
title: Mesos (1) -- Install Mesos on RHEL6
---

## Install dependencies:
Install maven manually:

    wget http://mirrors.cnnic.cn/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
    tar zxvf apache-maven-3.3.3-bin.tar.gz 
    mv apache-maven-3.3.3 /usr/local/apache-maven

Install 3rd libraries in RHEL

    yum install -y python-devel java-1.7.0-openjdk-devel zlib-devel libcurl-devel subversion-devel patch libtool

Install Mesos package:

    ./configure
    make && make install

Network configuration:

Proxies for Maven

vim $MAVEN_HOME/conf/settings.xml



    ...
    
    <proxy>
         <id>proxybj_http</id>
              <active>true</active>
              <protocol>http</protocol>
              <username></username>
              <password></password>
              <host>proxy_host</host>
              <port>8080</port>
              <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
    
    <proxy>
        <id>proxybj_https</id>
        <active>true</active>
        <protocol>https</protocol>
        <username></username>
        <password></password>
        <host>proxy_host</host>
        <port>8080</port>
        <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
    
    ...

Proxies for command line:

    export http_proxy=http://proxy_host:8080
    export https_proxy=http://proxy_host:8080

Install ZooKeeper:
Insall steps
Download zookeeper from offical site:

    wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz 

Configure zookeeper cluster for cua01/cua02/cua03:

    [root@cua01 zookeeper-3.4.6]# cp $ZK_HOME/conf/zoo_sample.cfg $ZK_HOME/conf/zoo.cfg
    [root@cua01 zookeeper-3.4.6]# cat conf/zoo.cfg 
    # The number of milliseconds of each tick
    tickTime=2000
    # The number of ticks that the initial 
    # synchronization phase can take
    initLimit=10
    # The number of ticks that can pass between 
    # sending a request and getting an acknowledgement
    syncLimit=5
    # the directory where the snapshot is stored.
    # do not use /tmp for storage, /tmp here is just 
    # example sakes.
    dataDir=/var/lib/zookeeper
    # the port at which the clients will connect
    clientPort=2181
    # the maximum number of client connections.
    # increase this if you need to handle more clients
    #maxClientCnxns=60
    # 
    # Be sure to read the maintenance section of the 
    # administrator guide before turning on autopurge.
    #
    # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
    #
    # The number of snapshots to retain in dataDir
    #autopurge.snapRetainCount=3
    # Purge task interval in hours
    # Set to "0" to disable auto purge feature
    #autopurge.purgeInterval=1
    server.1=cua01:2888:3888
    server.2=cua02:2888:3888
    server.3=cua03:2888:3888
    [root@cua01 zookeeper-3.4.6]# echo 1 > /var/lib/zookeeper/myid 
    [root@cua01 zookeeper-3.4.6]# cat /var/lib/zookeeper/myid     
    1

Start zookeeper on cua01/cua02/cua03:

    [root@cua01 zookeeper-3.4.6]# ./bin/zkServer.sh start
    JMX enabled by default
    Using config: /root/zookeeper-3.4.6/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED

Troubleshooting:
Exception during startup, resolved after all nodes were started

    2015-05-25 09:51:56,224 [myid:1] - WARN  [QuorumPeer[myid=1]/0:0:0:0:0:0:0:0:2181:QuorumCnxManager@382] - Cannot open channel to 2 at election address cua02/9.111.159.75:3888
    java.net.ConnectException: Connection refused
    at java.net.PlainSocketImpl.socketConnect(Native Method)
    at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:339)
    at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:200)    
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:182)
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
    at java.net.Socket.connect(Socket.java:579)
    at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:368)
    at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectAll(QuorumCnxManager.java:402)
    at org.apache.zookeeper.server.quorum.FastLeaderElection.lookForLeader(FastLeaderElection.java:840)
    at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:762)

Avoid duipicated localhost in /etc/hosts

    [root@cua01 zookeeper-3.4.6]# cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    127.0.0.1    cua01 #remove this line, or the zookeeper failed to bind leader port

    9.111.159.80    cua01
    9.111.159.75    cua02
    9.111.159.76    cua03
    9.111.159.77    cua04    

    [root@cua01 zookeeper-3.4.6]# cat zookeeper.out

    ...

    2015-05-24 17:35:40,439 [myid:1] - INFO  [Thread 1:QuorumCnxManager$Listener@504] - My election bind port: cua01/127.0.0.1:3888

    ...

Start Mesos

    nohup mesos-master --zk=zk://cua01:2181,cua02:2181,cua03:2181/mesos --log_dir=/root/mesos-0.22.1/log  --work_dir=/root/mesos-0.22.1/work/ --quorum=2 > /root/mesos-0.22.1/log/mesos.log &
    nohup mesos-slave --master=zk://cua01:2181,cua02:2181,cua03:2181/mesos --log_dir=/root/mesos-0.22.1/log  --work_dir=/root/mesos-0.22.1/work/ > /root/mesos-0.22.1/log/mesos.log &

Using web GUI at http://cua01:5050/ to check mesos cluster’s status.
