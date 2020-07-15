---
layout: post
title:  "How To Make A Multi Node Kafka Cluster"
date:   2020-07-18 21:35:20 +0200
categories: kafka zookeeper distributed-systems
---
### Introduction

Apache Kafka is one of those tools that you sure have heard about in a conference or from a coworker. Usually, is used as 
a Messaging System to decouple monoliths, but could be used as a Storage System or for Stream Processing too, in this tutorial 
I will show you how to create a multi-node kafka cluster with three different machines. 