# HADOOP CLUSTER DEPLOYMENT WITH DOCKER
This guide provides step-by-step instructions to initialize a Hadoop network, build images, and configure the Java environment within containers.

🏗️ 1. Infrastructure Setup
First, we establish the virtual network and build the base Hadoop image from the Dockerfile.

Bash

### Create a dedicated bridge network for Hadoop
```docker network create hadoop_network```

### Build the Hadoop image with tag 3.3.6
```docker build -t hadoop-base:3.3.6 .```

### Launch the cluster in detached mode
```docker compose up -d```

### Verify running containers
```docker ps```

### Access the NameNode container via bash
```docker exec -it namenode bash```
# 2. Environment & Java Configuration
Once inside the container, we must update the system and install the Java Development Kit (JDK 11).

### Update package lists and install essential utilities
```apt update && apt-get update```
```apt install nano wget -y```

### Download OpenJDK 11 binaries
```wget https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.9.1%2B1/OpenJDK11U-jdk_x64_linux_hotspot_11.0.9.1_1.tar.gz```

### Extract and set up the Java directory
```tar -xvf OpenJDK11U-jdk_x64_linux_hotspot_11.0.9.1_1.tar.gz -C /opt```
```mv /opt/jdk-11.0.9.1+1 /opt/jdk-11```

### Set up JAVA_HOME environment variables
```export JAVA_HOME=/opt/jdk-11```
```export PATH=$JAVA_HOME/bin:$PATH```

### Persist environment variables for future sessions
```echo 'export JAVA_HOME=/opt/jdk-11' >> ~/.bashrc```
```echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc```
```source ~/.bashrc```
### Hadoop Core Configuration
Following the Lab instructions (Section 3.1.1.3), configure the core XML files to define the NameNode and HDFS storage settings. 

### Core Site Configuration (core-site.xml)
Edit the file at $HADOOP_HOME/etc/hadoop/core-site.xml and add the following: 


```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://namenode:8020</value>
                <description>Where HDFS NameNode can be found on the network</description>
        </property>

        <property>
                <name>hadoop.http.staticuser.user</name>
                <value>root</value>
                <description>Whose can view the logs in the container application</description>
        </property>

        <property>
                <name>io.compression.codecs</name>
                <value>org.apache.hadoop.io.compress.SnappyCodec</value>
        </property>
</configuration>
```

### HDFS Configuration (hdfs-site.xml)
Define the replication factor and storage directories: 

```
<configuration>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>dfs.permissions.enabled</name>
                <value>false</value>
        </property>

        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>

        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/hadoop/dfs/name</value>
        </property>

        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/hadoop/dfs/data</value>
        </property>

        <property>
                <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
                <value>false</value>
        </property>

        <!-- Allow multihomed network for security, availability and performance-->
        <property>
                <name>dfs.namenode.rpc-bind-host</name>
                <value>0.0.0.0</value>
                <description>
                        controls what IP address the NameNode binds to.
                        0.0.0.0 means all available.
                </description>
        </property>

        <property>
                <name>dfs.namenode.servicerpc-bind-host</name>
                <value>0.0.0.0</value>
                <description>
                        controls what IP address the NameNode binds to.
                        0.0.0.0 means all available.
                </description>
        </property>

        <property>
                <name>dfs.namenode.http-bind-host</name>
                <value>0.0.0.0</value>
                <description>
                        controls what IP address the NameNode binds to.
                        0.0.0.0 means all available.
                </description>
        </property>

        <property>
                <name>dfs.namenode.https-bind-host</name>
                <value>0.0.0.0</value>
                <description>
                        controls what IP address the NameNode binds to.
                        0.0.0.0 means all available.
                </description>
        </property>

        <property>
                <name>dfs.client.use.datanode.hostname</name>
                <value>true</value>
                <description>
                        Whether clients should use datanode hostnames when
                        connecting to datanodes.
                </description>
        </property>

        <property>
                <name>dfs.datanode.use.datanode.hostname</name>
                <value>true</value>
                <description>
                        Whether datanodes should use datanode hostnames when
                        connecting to other datanodes for data transfer.
                </description>
        </property>
</configuration>
```

### mapred-site.xml
```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
                <description>
                        How hadoop execute the job, use yarn to execute the job
                </description>
        </property>

        <property>
                <name>mapred_child_java_opts</name>
                <value>-Xmx4096m</value>
        </property>

        <property>
                <name>mapreduce.map.memory.mb</name>
                <value>4096</value>
        </property>

        <property>
                <name>mapreduce.reduce.memory.mb</name>
                <value>8192</value>
        </property>

        <property>
                <name>mapreduce.map.java.opts</name>
                <value>-Xmx3072m</value>
        </property>

        <property>
                <name>mapreduce.reduce.java.opts</name>
                <value>-Xmx6144m</value>
        </property>

        <property>
                <name>yarn.app.mapreduce.am.env</name>
                <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.3.6/</value>
                <description>
                        Environment variable where MapReduce job will be processed
                </description>
        </property>

        <property>
                <name>mapreduce.map.env</name>
                <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.3.6/</value>
        </property>

        <property>
                <name>mapreduce.reduce.env</name>
                <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.3.6/</value>
        </property>

        <!-- Allow multihomed network for security, availability and performance-->
        <property>
                <name>yarn.nodemanager.bind-host</name>
                <value>0.0.0.0</value>
        </property>
</configuration>

Cấu hình file yarn-site.xml:
<configuration>
        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
                <description>
                        Log aggregation collects each container's logs and
                        moves these logs onto a file-system
                </description>
        </property>

        <property>
                <name>yarn.log.server.url</name>
                <value>http://historyserver:8188/applicationhistory/logs/</value>
                <description>
                        URL for log aggregation server
                </description>
        </property>

        <property>
                <name>yarn.resourcemanager.recovery.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.resourcemanager.store.class</name>
                <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore</value>
        </property>

        <property>
                <name>yarn.resourcemanager.scheduler.class</name>
                <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
        </property>

        <property>
                <name>yarn.scheduler.capacity.root.default.maximum-allocation-mb</name>
                <value>8192</value>
        </property>

        <property>
                <name>yarn.scheduler.capacity.root.default.maximum-allocation-vcores</name>
                <value>4</value>
        </property>

        <property>
                <name>yarn.resourcemanager.fs.state-store.uri</name>
                <value>/rmstate</value>
        </property>

        <property>
                <name>yarn.resourcemanager.system-metrics-publisher.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>resourcemanager</value>
        </property>

        <property>
                <name>yarn.resourcemanager.address</name>
                <value>resourcemanager:8032</value>
        </property>

        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>resourcemanager:8030</value>
        </property>

        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>resourcemanager:8031</value>
        </property>

        <property>
                <name>yarn.timeline-service.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.timeline-service.generic-application-history.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.timeline-service.hostname</name>
                <value>historyserver</value>
        </property>

        <property>
                <name>mapreduce.map.output.compress</name>
                <value>true</value>
        </property>

        <property>
                <name>mapred.map.output.compress.codec</name>
                <value>org.apache.hadoop.io.compress.SnappyCodec</value>
        </property>

        <property>
                <name>yarn.nodemanager.resource.memory-mb</name>
                <value>16384</value>
        </property>

        <property>
                <name>yarn.nodemanager.resource.cpu-vcores</name>
                <value>8</value>
        </property>

        <property>
                <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
                <value>98.5</value>
        </property>

        <property>
                <name>yarn.nodemanager.remote-app-log-dir</name>
                <value>/app-logs</value>
        </property>

        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>

        <property>
                <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>

        <!-- Allow multihomed network for security, availability and performance-->
        <property>
                <name>yarn.resourcemanager.bind-host</name>
                <value>0.0.0.0</value>
        </property>

        <property>
                <name>yarn.nodemanager.bind-host</name>
                <value>0.0.0.0</value>
        </property>

        <property>
                <name>yarn.timeline-service.bind-host</name>
                <value>0.0.0.0</value>
        </property>
</configuration>
```
# System Initialization
After configuration, format the NameNode and start the Hadoop services.  
### Format the HDFS file system (Perform only once)
```hdfs namenode -format```

### Start HDFS and YARN services
```start-dfs.sh```
```start-yarn.sh```



##  Hive Configuration
### Installation and Environment Setup
Download the Hive 4.0.1 installation package:

```wget [https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz](https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz)```
```tar -xzf apache-hive-4.0.1-bin.tar.gz```
```mv apache-hive-4.0.1-bin hive```

Configure environment variables in ~/.bashrc:
```nano ~/.bashrc```

Add the following lines to the end of the file:
```
export HADOOP_HOME=/opt/hadoop-3.6
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HIVE_HOME=/hive
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib/*:$HIVE_HOME/lib/*
source ~/.bashrc
```
### Configure hive-env.sh
```
cd hive/conf
cp hive-env.sh.template hive-env.sh
nano hive-env.sh
export HADOOP_HOME=/opt/hadoop-3.3.6
```
### 1.2.3 Configure hive-site.xml
```
cd hive/conf
cp hive-default.xml.template hive-site.xml
nano hive-site.xml
```
```
<property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/tmp/hive/hadoop-3.3.6/operation_logs</value>
</property>

<property>
    <name>hive.querylog.location</name>
    <value>/tmp/hive/hadoop-3.3.6</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
</property>

<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
</property>
```
### Create Hive Warehouse Directories on HDFS

```
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hive
hdfs dfs -mkdir /user/hive/warehouse
hdfs dfs -mkdir /user/tmp
hdfs dfs -chmod g+w /user/hive/warehouse
hdfs dfs -chmod g+w /user/tmp
```
### Initialize Hive Metastore Derby Database
```
schematool -initSchema -dbType derby
```
### Connect to Hive
```
hive
```

# Spark Configuration
### Download and Extract Spark
First, download the Spark package, extract the files, and move them to the system directory:

```wget https://dlcdn.apache.org/spark/spark-4.0.1/spark-4.0.1-bin-hadoop3.tgz```

```tar -xzf spark-4.0.1-bin-hadoop3.tgz```

```mv spark-4.0.1-bin-hadoop3 /spark```

### Configure Environment Variables
To make Spark commands accessible from any directory, update your bash profile:

Open the bash configuration file: ```nano ~/.bashrc```

Add the following environment variables:```export SPARK_HOME=/spark export PATH=$PATH:$HADOOP_HOME/bin:$HIVE_HOME/bin:$SPARK_HOME/bin```

Save the file and apply the changes: ```source ~/.bashrc```

### Edit spark-env.sh File

Navigate to the configuration folder and create the environment script from the template:

```cd /spark/conf```

```cp spark-env.sh.template spark-env.sh```

```nano spark-env.sh```

Start Spark

Once the configuration is complete, you can launch the Spark interactive shell by running:

```spark-shell```
