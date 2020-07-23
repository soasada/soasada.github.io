---
layout: post
title:  "How To Make A Multi Node Kafka Cluster"
date:   2020-07-18 21:35:20 +0200
categories: kafka zookeeper distributed-systems
---
### Introduction

Apache Kafka (from now on just Kafka) is one of the most frequently used message buses nowadays, and the reason is because 
has some key features that makes it the perfect tool in some scenarios. Usually, is used as a Messaging System (for microservices 
architectures, decoupling monoliths, etc), but could even be used as a Storage System or for Stream Processing. 

In this tutorial I will show you how to create a multi-node Kafka cluster with three different machines. Although, this tutorial is for administrators 
I think that is worth read for those that use Kafka in his daily development. 

Recently, for one of my side projects I had the need of create a totally different database from another one, but in insertion order, 
I mean when there is an insertion in database A must be another in the same order in database B but the data coming from 
A should be enriched before inserted in B. To do it, I choose Kafka since it allows to consume messages in order (per partition) and 
to have fault tolerance a cluster is a must. The cluster has the following anatomy: 

![Kafka Cluster](/assets/kafka_cluster.png)

As usual, one of the server is going to act as the "leader" and the others as the "followers". If the leader stop running 
one of the followers will automatically become the new leader.

Knowing that, we have to carefully configure each broker with the correct parameters in order to fill our needs in the way we want. 
For the time I'm writting this the last version of Kafka is 2.5.0 and is the one I'm going to use here (**kafka_2.13-2.5.0.tgz**).

### Make a ZooKeeper cluster

Kafka use ZooKeeper to persist its metadata: location of partitions or configuration of the topics (eventually this dependency 
will be removed [Confluent post talking about it](https://www.confluent.io/blog/removing-zookeeper-dependency-in-kafka/)). So first, we need 
to create a ZooKeeper cluster (`ensemble` how is called by ZooKeeper). There is no need to download ZooKeeper because it's bundled with Kafka.

I initialize a git repository within the `config` folder inside the Kafka main folder, this gives me two advantages (among others):

1. I have available the config in all servers.
2. I will track the changes in config if needed.

Now, edit `zookeeper.properties` file inside `config` folder with the following:

{% highlight bash %}
tickTime=2000
dataDir=/path/you/want # be sure you have write permissions
clientPort=2181
initLimit=5
syncLimit=2
server.1=<IP1>:2888:3888 # every machine that is part of
server.2=<IP2>:2888:3888 # the ZooKeeper cluster should 
server.3=<IP3>:2888:3888 # know about each other.
{% endhighlight %}

The explanation of each option is (extracted from docs): 

**tickTime:** the length of a single tick, which is the basic time unit used by ZooKeeper, as measured in milliseconds. 
It is used to regulate heartbeats, and timeouts. For example, the minimum session timeout will be two ticks. 

**dataDir:** the location where ZooKeeper will store the in-memory database snapshots and, unless specified otherwise, the 
transaction log of updates to the database. 

**clientPort:** the port to listen for client connections; that is, the port that clients attempt to connect to. 

**initLimit:** (No Java system property) Amount of time, in ticks (see tickTime), to allow followers to connect and sync to a 
leader. Increased this value as needed, if the amount of data managed by ZooKeeper is large. 

**syncLimit:** (No Java system property) Amount of time, in ticks (see tickTime), to allow followers to sync with ZooKeeper. 
If followers fall too far behind a leader, they will be dropped. 

**server.x=[hostname]:nnnnn[:nnnnn], etc:** (No Java system property) servers making up the ZooKeeper ensemble. When the server starts up, it determines 
which server it is by looking for the file myid in the data directory. That file contains the server number, in ASCII, 
and it should match x in server.x in the left hand side of this setting. The list of servers that make up ZooKeeper servers 
that is used by the clients must match the list of ZooKeeper servers that each ZooKeeper server has. There are two port numbers nnnnn. 
The first followers use to connect to the leader, and the second is for leader election. If you want to test multiple servers on a single machine, 
then different ports can be used for each server.

Also, you need to create a file called `myid` inside the `dataDir` you specified with just the number of the server, for 
my case I will have three `myid` files one in each server with just "1", "2" and "3".

Then, make a copy from `zookeeper.properties` for each machine:

{% highlight bash %}
cp zookeeper.properties zookeeper2.properties
cp zookeeper.properties zookeeper3.properties
{% endhighlight %}

...and change the `dataDir` for each one, now you could launch each ZooKeeper server in each machine as follows (inside Kafka folder):

{% highlight bash %}
bin/zookeeper-server-start.sh config/zookeeper.properties &
bin/zookeeper-server-start.sh config/zookeeper2.properties &
bin/zookeeper-server-start.sh config/zookeeper3.properties &
{% endhighlight %}

If you execute `jps -l` you should see something like the following:

{% highlight bash %}
4327 org.apache.zookeeper.server.quorum.QuorumPeerMain
4825 jdk.jcmd/sun.tools.jps.Jps
{% endhighlight %}

...which means that you have running your ZooKeeper server.

### References

[Kafka Documentation](https://kafka.apache.org/25/documentation.html)
[ZooKeeper Documentation](https://zookeeper.apache.org/doc/r3.5.7/)