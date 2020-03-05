# Setup Apache Hadoop Cluster on EC2 (Ubuntu 18.04.3 LTS)


## Set up
This repo is to document/ share the process of setting up Hadoop v3.2.1 Cluster on AWS EC2, even though there are already cloud providers provde this service (such as Amazon's EMR and GCP's Dataproc)

0. Assumed you already have created 4 (1 namenode + 3 datanodes) instances (which the operating system is Ubuntu 18.04.3 LTS)

1. Set up environments [for all 4 nodes]
```
# update os
sudo apt-get update && sudo apt-get upgrade

# install java 8
sudo apt-get -y install openjdk-8-jdk-headless

# install Apache Hadoop 3.2.1
wget http://apache.mirrors.hoobly.com/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
tar xvzf hadoop-3.2.1.tar.gz
mv ./hadoop-3.2.1 /usr/local/hadoop-3.2.1
```

2. Environment variables set up [for all 4 nodes]

- Edit ~/.bashrc by adding the following Hadoop variables
```
### HADOOP ENVIRONMENT VARIABLES START ###
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop-3.2.1
export PATH=$PATH:${HADOOP_HOME}/bin
export PATH=$PATH:${HADOOP_HOME}/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_CONF_DIR=${HADOOP_HOME/etc/hadoop
export HADOOP_PREFIX=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_HOME}/lib/native
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
### HADOOP ENVIRONMENT VARIABLES END ###
```

after closing ~/.bashrc, source the file
```
source ~/.bashrc
```

- Edit $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
# Set up JAVA_HOME for all nodes
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

3.1 Create directory on Namenode
```
sudo mkdir -p /usr/local/hadoop/data/hdfs/namenode
# change owner of the directory to root(ubuntu)
sudo chown -R ubuntu /usr/local/hadoop
```

3.2 Create directory on each Datanode
```
sudo mkdir -p /usr/local/hadoop/data/hdfs/datanode
# change owner of the directory to root(ubuntu)
sudo chown -R ubuntu /usr/local/hadoop
```


4. Setup SSH Config on NameNode
Edit ~/.ssh/config 
```
Host namenode
  HostName ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:
  User ubuntu
  IdentityFile ~/.ssh/id_rsa

Host datanode1
  HostName ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:
  User ubuntu
  IdentityFile ~/.ssh/id_rsa

Host datanode2
  HostName ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:
  User ubuntu
  IdentityFile ~/.ssh/id_rsa

Host datanode3
  HostName ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:
  User ubuntu
  IdentityFile ~/.ssh/id_rsa
```

5. Add nodes' private IP address to /etc/hosts file on NameNode
```
10.x.x.x namenode
10.x.x.x datanode1
10.x.x.x datanode2
10.x.x.x datanode3
```

6. Configure $HADOOP_HOME/etc/Hadoop/workers on NameNode by adding
```
datanode1
datanode2
datanode3
```

7. Create a new file named "masters" under $HADOOP_HOME/etc/hadoop/ on NameNode
Add namenode's public DNS
```
ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com
```

8. Update five Hadoop configuration files on Namenode
[we will use bash script to copy the files to datanodes later]
- Add the following to $HADOOP_HOME/etc/hadoop/core-site.xml
```
<configuration> 
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:9000</value>
  </property>
 </configuration>
```

- Add the following to $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:///usr/local/hadoop/data/hdfs/namenode</value>
    </property>

    <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:///usr/local/hadoop/data/hdfs/datanode</value>
    </property>

    <property>
            <name>dfs.replication</name>
            <value>2</value>
    </property>
</configuration>
```

- Add the following to $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
<configuration>
  <property>
           <name>mapreduce.jobtracker.address</name>
           <value>ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com:54311</value>
  </property>
  <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
  </property>
  <property>
          <name>yarn.app.mapreduce.am.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property>
  <property>
          <name>mapreduce.map.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property>
  <property>
          <name>mapreduce.reduce.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property> </configuration>
```

- Add the following to $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
<configuration>

<!-- Site specific YARN configuration properties -->

    <property>
            <name>yarn.acl.enable</name>
            <value>0</value>
    </property>

    <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com
</value>
    </property>

    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>

</configuration>
```

8. Copy the Hadoop configuration files to the three datanodes through
typing the below commands on Namenode.
```
for node in dnode1 dnode2 dnode3; do
scp $HADOOP_HOME/etc/hadoop/* $node:/$HADOOP_HOME/etc/hadoop/;
done
```
# courtesy: https://www.linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/


## Testing the cluster

Now start the HDFS cluster by running the following on namenode.
```
# format the namenode
hdfs namenode -format

HADOOP_HOME/sbin/start-dfs.sh
# check the running process
jps
```

Now you should be seeing the following. If any is missing, it means you do not set up the cluster correctly.
```
xxxx Jps
xxxx NameNode
xxxx SecondaryNameNode
```

p.s. on datanode, when you run `jps`, you shoud see the following
```

```

start the Yarn resource manager on namenode.
```
HADOOP_HOME/sbin/start-yarn.sh
# check the running process
jps
```

Now you shoud see the following on namenode
```
xxxx Jps
xxxx NameNode
xxxx SecondaryNameNode
xxxx ResourceManager
```
a  and on the datanodes, after running `jps` you'll see
```
xxxx DataNode
xxxx Jps
xxxx NodeManager
```

## Trouble Shooting

1.
```
If not seeing any DataNode, NameNode, SecondaryNameNode, ResouceManager, DataMnager, then check $HADOOP_HOME/log/* first to see what happened.
```

2.
Error log as
```
Hadoop job stuck on ACCEPTED when there is a spark job RUNNING
yarn.scheduler.capacity.maximum-am-resource-percent
```
Whe you see the above error log, it means your namenode does not have enough memory. You'd need to tune the default settings or re-create new instances with more memory capacity.

3.
```
If seeing “hadoop java.net.BindException: Port in use”
```
Use the script to kill the process `sudo kill -9 $PID`
PID can be found through the command `sudo lsof -i :$PORT`
* Namenode ports are
- 9868
- 8088
* Datanode ports are
- 9866
- 8040
