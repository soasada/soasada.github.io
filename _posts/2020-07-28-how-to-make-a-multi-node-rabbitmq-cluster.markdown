---
layout: post
title:  "How To Make A Multi Node RabbitMQ Cluster"
date:   2020-07-28 20:02:07 +0200
categories: rabbitmq distributed-systems
---
### 1. Introduction

Several days ago, [I made a tutorial](/kafka/zookeeper/distributed-systems/2020/07/18/how-to-make-a-multi-node-kafka-cluster.html) 
of how to install Apache Kafka in a multi node cluster, what's wrong with that? Well, if you have cheap VPS machines like 
me, maybe you don't want to install Kafka due to the high hardware requirements it needs (and ZooKeeper) or because you want 
to use RabbitMQ over Apache Kafka. 

Whatever the reason you have, I'm going to create another tutorial to guide you (and me in the future) through the installation 
of a multi node cluster of RabbitMQ.

For this tutorial I'm going to use Ubuntu 18.04 as OS in each machine and two machines for the cluster.

### 2. Preparing the machines

The first thing that you have to do, is to configure the hostname of each machine. To do it, you have to follow the next steps 
in every machine of the cluster: 

#### 2.1 Disable the cloud-init module (Optional)

Assuming that your VPS is in some cloud provider (I'm using OVH), maybe you have to disable the cloud-init module that 
overwrites the `/etc/hosts` file in every system reboot. This could lead to problems if you don't disable it. To 
do it so, you have to:

* Edit the cloud-init configuration file:

{% highlight bash %}
sudo vim /etc/cloud/cloud.cfg
{% endhighlight %}

* Add or modify the following two lines:

{% highlight bash %}
preserve_hostname: true
manage_etc_hosts: false
{% endhighlight %}

...save and exit.

#### 2.2 Modify your hostname (Optional)

If you have a domain name pointing to your machine, you probably want to put it as hostname. 

* Modify your hostname:

{% highlight bash %}
sudo vim /etc/hostname
{% endhighlight %}

* Put your domain name without the domain:

{% highlight bash %}
mydomain
{% endhighlight %}

...save and exit.

* Modify `/etc/hosts` file:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Put your IP:

{% highlight bash %}
127.0.0.1         localhost
XXX.XXX.XXX.XXX   mydomain.com    mydomain
127.0.1.1         mydomain.com    mydomain
{% endhighlight %}

...save and exit. 

* Reboot the system:

{% highlight bash %}
sudo reboot
{% endhighlight %}
 
* Check if everything is still there:

{% highlight bash %}
sudo cat /etc/hosts
{% endhighlight %}

...you should see you config:

{% highlight bash %}
127.0.0.1         localhost
XXX.XXX.XXX.XXX   mydomain.com    mydomain
127.0.1.1         mydomain.com    mydomain
{% endhighlight %}

* Check if the hostname is correct:

{% highlight bash %}
hostname -f
{% endhighlight %}

...should print:

{% highlight bash %}
mydomain.com
{% endhighlight %}

#### 2.3 Add other machine host

For each of the machines, edit your `/etc/hosts` and add the IP and hostname of the other.

* In machine1:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Add the IP and hostname of the other machine:

{% highlight bash %}
YYY.YYY.YYY.YYY mydomain2 # IP and hostname of the machine2
{% endhighlight %}

* In machine2:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Add the IP and hostname of the other machine:

{% highlight bash %}
XXX.XXX.XXX.XXX mydomain1 # IP and hostname of the machine1
{% endhighlight %}

### 3. Installing RabbitMQ

In every machine, you have to install RabbitMQ. I will use the apt package that RabbitMQ provides to install RabbitMQ.

{% highlight bash %}
#!/usr/bin/env bash

sudo apt update

sudo apt install curl gnupg -y

curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -

sudo apt install apt-transport-https

sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF

sudo apt update

sudo apt install rabbitmq-server -y --fix-missing
{% endhighlight %}

This script installs the latest version of RabbitMQ along with the latest Erlang version. Now you have to create a config 
file for RabbitMQ:

{% highlight bash %}
sudo vim /etc/rabbitmq/rabbitmq.conf
{% endhighlight %}

With the following content:

{% highlight bash %}
listeners.tcp.default = localhost:5672
{% endhighlight %}

For security reasons we should keep each RabbitMQ instance with the AMQP port listening in localhost, we will configure a reverse proxy with Nginx 
and add SSL Termination on top of it. Additionally, RabbitMQ have a port listening in all interfaces for inter-node communication, `25672` by default, 
you could also put it behind the proxy, but I will keep it listening in all interfaces. 

If everything went well, you should see the RabbitMQ server running: 

{% highlight bash %}
sudo service rabbitmq-server status
{% endhighlight %}

If not, you could start it with:

{% highlight bash %}
sudo service rabbitmq-server start
{% endhighlight %}

Also, you could see the logs in:

{% highlight bash %}
cat /var/log/rabbitmq/rabbit@mydomain.log
{% endhighlight %}

### 4. Creating the cluster

For each node execute:

{% highlight bash %}
sudo rabbitmq-server -detached
{% endhighlight %}

### 5. SSL Termination with Let's Encrypt and Nginx

### Conclusion

### References

1. [OVH disable cloud-init for hostname](https://docs.ovh.com/gb/en/public-cloud/changing_the_hostname_of_an_instance/)
2. [RabbitMQ installation](https://www.rabbitmq.com/install-debian.html)