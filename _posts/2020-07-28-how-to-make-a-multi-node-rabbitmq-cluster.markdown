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

For this tutorial I'm going to use Ubuntu 18.04 as OS in each machine and three machines for the cluster.

### 2. Preparing the machines

The first thing that you have to do, is to configure the hostname of each machine. RabbitMQ will use the hostname of the machines 
to communicate with each other. To do it, **you have to follow the next steps in every machine of the cluster.** 

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
mydomain1
{% endhighlight %}

...save and exit.

* Modify `/etc/hosts` file:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Put your IP:

{% highlight bash %}
127.0.0.1         localhost
XXX.XXX.XXX.XXX   mydomain1.com    mydomain1
127.0.1.1         mydomain1.com    mydomain1
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
XXX.XXX.XXX.XXX   mydomain1.com    mydomain1
127.0.1.1         mydomain1.com    mydomain1
{% endhighlight %}

* Check if the hostname is correct:

{% highlight bash %}
hostname -f
{% endhighlight %}

...should print:

{% highlight bash %}
mydomain1.com
{% endhighlight %}

#### 2.3 Add other machine host

For each of the machines, edit your `/etc/hosts` and add the IP and hostname of the other.

* In machine1:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Add the IP and hostname of the other machines:

{% highlight bash %}
YYY.YYY.YYY.YYY mydomain2.com mydomain2 # IP and hostname of the machine2
ZZZ.ZZZ.ZZZ.ZZZ mydomain3.com mydomain3 # IP and hostname of the machine3
{% endhighlight %}

* In machine2:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Add the IP and hostname of the other machines:

{% highlight bash %}
XXX.XXX.XXX.XXX mydomain1.com mydomain1 # IP and hostname of the machine1
ZZZ.ZZZ.ZZZ.ZZZ mydomain3.com mydomain3 # IP and hostname of the machine3
{% endhighlight %}

* In machine3:

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

* Add the IP and hostname of the other machines:

{% highlight bash %}
XXX.XXX.XXX.XXX mydomain1.com mydomain1 # IP and hostname of the machine1
YYY.YYY.YYY.YYY mydomain2.com mydomain2 # IP and hostname of the machine2
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
listeners.tcp.default = 127.0.0.1:5672
{% endhighlight %}

For security reasons we should keep each RabbitMQ instance with the AMQP port listening in localhost, we will configure a reverse proxy with Nginx 
and add SSL Termination on top of it. Additionally, RabbitMQ have a port listening in all interfaces for inter-node communication, `25672` by default, 
you could also put it behind the proxy, but I will keep it listening in all interfaces. 

If everything went well, you should see the RabbitMQ server running: 

{% highlight bash %}
sudo service rabbitmq-server status # or rabbitmq-diagnostics status
{% endhighlight %}

If not, you could start it with:

{% highlight bash %}
sudo service rabbitmq-server start
{% endhighlight %}

Also, you could see the logs in:

{% highlight bash %}
cat /var/log/rabbitmq/rabbit@mydomain1.log
{% endhighlight %}

### 4. Creating the cluster

Now we can setup the cluster.

#### 4.1 Setting the Erlang cookie

In order to connect each node with the main one, you should use the Erlang cookie. Usually, it is located at:

{% highlight bash %}
/var/lib/rabbitmq/.erlang.cookie
{% endhighlight %}

...in the main machine (mydomain1 node in my case). If not, you could use the following command that tells you the information 
about the cookie:

{% highlight bash %}
rabbitmq-diagnostics erlang_cookie_sources
{% endhighlight %}

...the output is:

{% highlight bash %}
Listing Erlang cookie sources used by CLI tools...
Cookie File

Effective user: rabbitmq
Effective home directory: /var/lib/rabbitmq
Cookie file path: /var/lib/rabbitmq/.erlang.cookie
Cookie file exists? true
Cookie file type: regular
Cookie file access: read
Cookie file size: 20

Cookie CLI Switch

--erlang-cookie value set? false
--erlang-cookie value length: 0

Env variable  (Deprecated)

RABBITMQ_ERLANG_COOKIE value set? false
RABBITMQ_ERLANG_COOKIE value length: 0
{% endhighlight %}

Then get the cookie:

{% highlight bash %}
cat /var/lib/rabbitmq/.erlang.cookie
{% endhighlight %}

...and copy/paste it in mydomain2 and mydomain3 nodes in the same file.

#### 4.2 Join mydomain2 and mydomain3 to mydomain1 cluster

For each node execute:

{% highlight bash %}
rabbitmq-server -detached # in mydomain1, mydomain2 and mydomain3
{% endhighlight %}

...this creates three RabbitMQ brokers that don't know nothing of each other. Now let's say that we are gonna use mydomain1 node as the main one, 
we have to tell to mydomain2 and mydomain3 nodes to join the cluster. To do that, on mydomain2 we have to stop the RabbitMQ 
application and join the mydomain1 cluster, then restart the RabbitMQ application.

On mydomain2 and mydomain3 execute:

{% highlight bash %}
rabbitmqctl stop_app
{% endhighlight %}

...then reset the RabbitMQ application:

{% highlight bash %}
rabbitmqctl reset
{% endhighlight %}

...tell to join the cluster:

{% highlight bash %}
rabbitmqctl join_cluster rabbit@mydomain1
{% endhighlight %}

...and start again the RabbitMQ application:

{% highlight bash %}
rabbitmqctl start_app
{% endhighlight %}

To check that all nodes have joined to the cluster execute in each node:

{% highlight bash %}
rabbitmqctl cluster_status
{% endhighlight %}

...you should see something like:

{% highlight bash %}
Cluster status of node rabbit@mydomain1 ...
[{nodes,[{disc,[rabbit@mydomain1,rabbit@mydomain2,rabbit@mydomain3]}]},
{running_nodes,[rabbit@mydomain3,rabbit@mydomain2,rabbit@mydomain1]}]
...done.
{% endhighlight %}

### 5. SSL Termination with Let's Encrypt and Nginx

Assuming you have already a certificate issued by Let's Encrypt I will add SSL termination to every node. To do that, 
I'm going to use the stream module of Nginx (I also assume that you have Nginx in every machine).

In every machine, copy/paste the following config:

{% highlight bash %}
stream {
        upstream rabbitmq {
                server localhost:5672;
        }

        server {
                listen                5671 ssl;
                proxy_pass            rabbitmq;

                ssl_certificate /etc/letsencrypt/live/mydomain1.com/fullchain.pem; # change to mydomain2 and mydomain3
                ssl_certificate_key /etc/letsencrypt/live/mydomain1.com/privkey.pem; # change to mydomain2 and mydomain3

                ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

                ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
                ssl_session_cache     shared:SSL:20m;
                ssl_session_timeout   4h;
                ssl_handshake_timeout 30s;
                ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS";
        }
}
{% endhighlight %}

Note that now every client have to connect to the cluster with the following string:

{% highlight bash %}
"XXX.XXX.XXX.XXX:5671,YYY.YYY.YYY.YYY:5671,ZZZ.ZZZ.ZZZ.ZZZ:5671"
{% endhighlight %}

### Conclusion

RabbitMQ could have some problems comparing with Kafka, for example your client could not be connected automatically to another 
node if one fails (you should ensure that you client have the reconnection feature) or for example does not maintain the 
message ordering (if you have a queue per consumer it is guaranteed, if you have multiple consumers in parallel the order is not guaranteed).

Apart from that, RabbitMQ fits perfectly on my machines and consume less resources, I will try it and see if could be used 
for my needs, also installing it was a little bit more difficult than the Kafka cluster but the documentation of RabbitMQ is 
quite good, better than the Kafka one.

### References

1. [OVH disable cloud-init for hostname](https://docs.ovh.com/gb/en/public-cloud/changing_the_hostname_of_an_instance/)
2. [RabbitMQ installation](https://www.rabbitmq.com/install-debian.html)
2. [RabbitMQ clustering](https://www.rabbitmq.com/clustering.html)
3. [Nginx SSL termination docs](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-tcp/)