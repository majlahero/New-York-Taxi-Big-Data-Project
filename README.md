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
        <value>hdfs://master:9000</value>
    </property>
</configuration>
```

### HDFS Configuration (hdfs-site.xml)
Define the replication factor and storage directories: 

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hdoop/hadoopdata/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hdoop/hadoopdata/hdfs/datanode</value>
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

### Verify active processes
```jps```
