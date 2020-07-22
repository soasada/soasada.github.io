---
layout: post
title:  "How To Make A Multi Node Kafka Cluster"
date:   2020-07-18 21:35:20 +0200
categories: kafka zookeeper distributed-systems
---
### Introduction

Apache Kafka (from now on just Kafka) is one of those tools that you have heard about it, in a conference or from a coworker. 
Usually, is used as a Messaging System (for microservices architectures, decoupling monoliths, etc), but could be used as a 
Storage System or for Stream Processing also, in this tutorial I will show you how to create a multi-node kafka cluster with 
three different machines. Although, this tutorial is for administrators I think that is worth read for those that use kafka 
in his daily development. 

Recently, for one of my side projects I had the need of create a totally different database from another one, using the existing 
data but adding some other stuff. To do it, I choose to build a Kafka cluster first and then produce and consume this data 
from one database to another. In order to have high availability and to not lose data in the migration I decide to create 
a cluster with three distributed Kafka brokers.

### Make a Zookeeper cluster

Kafka use Zookeeper for broker coordination, first 