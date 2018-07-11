## Run Hadoop Cluster within Docker Containers

- Reference: [Run Hadoop Cluster in Docker Update](http://kiwenlau.com/2016/06/26/hadoop-cluster-docker-update-english/)


![alt tag](https://raw.githubusercontent.com/princhenee/hadoop-cluster-docker/master/hadoop_cluster_topo.png)


### Tri-Nodes Hadoop Cluster

##### 1. pull docker image

```
sudo docker pull kiwenlau/hadoop:1.0
```

##### 2. clone github repository

```
git clone https://github.com/princhenee/hadoop-cluster-docker
```

##### 3. create hadoop network

```
sudo docker network create --driver=bridge hadoop
```

##### 4. start container

```
cd hadoop-cluster-docker
sudo ./start-container.sh
```

**output:**

```
start hadoop-master container...
start hadoop-slave1 container...
start hadoop-slave2 container...
root@hadoop-master:~# 
```
- start 3 containers with 1 master and 2 slaves
- you will get into the /root directory of hadoop-master container

##### 5. start hadoop

```
./start-hadoop.sh
```

##### 6. run wordcount

```
./run-wordcount.sh
```

### Arbitrary size Hadoop cluster

##### 1. pull docker images and clone github repository

do 1~3 like section A

##### 2. rebuild docker image

```
sudo ./resize-cluster.sh 5
```
- specify parameter > 1: 2, 3..
- this script just rebuild hadoop image with different **slaves** file, which pecifies the name of all slave nodes


##### 3. start container

```
sudo ./start-container.sh 5
```
- use the same parameter as the step 2

##### 4. run hadoop cluster 

do 5~6 like section A

### Hadoop Cluster Monitor
#### 1. When the cluster is running, execute to check tcp connections established by java processes alive:
```
netstat -plten | grep java
```
**output**
```
tcp        0      0 172.18.0.4:8033         0.0.0.0:*               LISTEN      0          57506172    619/java
tcp        0      0 172.18.0.4:9000         0.0.0.0:*               LISTEN      0          57498947    199/java
tcp        0      0 0.0.0.0:50090           0.0.0.0:*               LISTEN      0          57489721    420/java
tcp        0      0 0.0.0.0:50070           0.0.0.0:*               LISTEN      0          57498944    199/java
tcp        0      0 172.18.0.4:8088         0.0.0.0:*               LISTEN      0          57482952    619/java
tcp        0      0 172.18.0.4:8030         0.0.0.0:*               LISTEN      0          57368571    619/java
tcp        0      0 172.18.0.4:8031         0.0.0.0:*               LISTEN      0          57368565    619/java
tcp        0      0 172.18.0.4:8032         0.0.0.0:*               LISTEN      0          57368575    619/java
```

#### 2. Listen to ports that related to dedicated processes
| Daemon        | Default Port  |     Protocol       |    Used For    |
| ------------- |---------------| ------------------ | -------------- |
| Namenode      | 8020          |IPC: `ClientProtocol` |  Filesystem metadata operations |
| Datanode      | 50010         |Custom Hadoop Receiver: `Datanode` and `DFSClient` | DFS data transfer |
| Datanode      | 50020         |IPC: `InterDatanodeProtocol`, `ClientDatanodeProtocol` | Block metadata operations and recovery |

run tcpdump to capture packets while the application (wordcount) is running.
```
$sudo docker exec -it hadoop-master bash
#tcpdump -i eth0 "port 50010" -X > port-50010.txt
```
output
```
the "port-50010.txt" file in the working directory
```


### Hadoop I/O
#### Serialization

Serialization is the process of turning structured objects into a byte stream for transmission over a network or for writing to persistent storage. Deserialization is the reverse process of turning a byte stream back into a series of structured objects.

In Hadoop, interprocess communication between nodes in the system is implemented using remote procedure calls (RPCs). The RPC protocol uses serialization to render the message into a binary stream to be sent to the remote node, which then deserializes the binary stream into the original message.

Hadoop uses its own serialization format, Writables, which is certainly compact and fast, but not so easy to extend or use from languages other than Java. Writables are central to Hadoop as most MapReduce programs use them for their key and value types.

![alt tag](https://raw.githubusercontent.com/princhenee/hadoop-cluster-docker/master/hadoop_writable_hierarchy.png)
