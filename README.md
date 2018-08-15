# OpenBMP docker files
Docker files for OpenBMP.

(Prerequisite) Platform Docker Install
--------------------------------------

> Ignore this step if you already have a current docker install

> ####NOTE
> You should use the latest docker version, documented in this section.

### CentOS 6

> #### CentOS 7 works fine by following the docker install instructions, so they are not documented here. 

The below are steps for how to install docker on CentOS 6.

```sh
# Add repo for docker package.  This works with CentOS 6.7 too
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

# Define any proxies if you have them
export http_proxy=https://proxy.blah.com:80
export https_proxy=https://proxy.blah.com:80

# Install docker
yum -y install docker-io

# add the below to /etc/sysconfig/docker if you have proxies
export http_proxy="http://proxy.blah.com:80"
export https_proxy="http://proxy.blah.com:80"

# Start docker
service docker start 
```

### Ubuntu 14.04 (trusty)
The below instructions are for Ubuntu 14.04, but you can install docker on CentOS or
any other distro/platform.  Follow the [Docker Install Instructions](http://docs.docker.com/installation/) for your distro/platform. 


> The below commands need to be run as '**root**', which can be done via '**sudo**'

    apt-get update
    apt-get install -y wget   
    
    apt-get install linux-image-extra-$(uname -r)
    modprobe aufs
    
    wget -qO- https://get.docker.com/ | sh
    
    # Optionally add a non-root user to run docker as
    usermod -aG docker ubuntu

    # Logout and log back so the group takes affect. 
    

Optionally configure **/etc/default/docker** (e.g. for proxy config)

    export http_proxy="http://proxy:80/"
    export https_proxy="http://proxy:80/"
    export no_proxy="127.0.0.1,openbmp.org,/var/run/docker.sock"

Make sure you can run '**docker run hello-world**' successfully.

```
ubuntu@docker:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
511136ea3c5a: Pull complete 
31cbccb51277: Pull complete 
e45a5af57b00: Pull complete 
hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Status: Downloaded newer image for hello-world:latest
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
```


Install OpenBMP using Docker
----------------------------
Each docker file contains a readme file, see below:

* [All-In-One](aio/README.md)
* [Collector](collector/README.md)
* [Kafka](kafka/README.md)
* [PostgreSQL](postgres/README.md)
* [MySQL](mysql/README.md)

> **openbmp/ui** is archived. Use [obmp-grafana](https://github.com/OpenBMP/obmp-grafana) instead.


Install OpenBMP using docker-compose
----------------------------
As alternative to [All In One](aio/README.md), docker image Collector, Kafka and Mysql can be started up using [docker-compose](https://docs.docker.com/compose/install/)

```
docker-compose up
```
