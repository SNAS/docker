OpenBMP Kafka Container
----------------------------
Kafka container is a pre-configured install for OpenBMP.  You can use this container for OpenBMP
or you can use your own Kafka install.  OpenBMP supports consumer load balancing, so we do
define at least 4 partitions.

#### Container Includes
* **Apache Kafka 0.8.x** - High performing message bus (listening ports are TCP 2181 and 9092)


### Recommended Current Linux Distributions

  1. Ubuntu 14.04/Trusty
  1. CentOS 7/RHEL 7

### 1) Install docker
Docker host should be **Linux x86_64**.   Follow the [Docker Instructions](https://docs.docker.com/installation/) to install docker.  

- - -

### 2) Download the docker image

    docker pull openbmp/kafka

- - -

### 3) Create Kafka persistent volume
Depending on your docker [devicedriver](https://docs.docker.com/engine/reference/commandline/dockerd/), the root filesystem in the container may not be the best
place to house the kafka data files.  The default for **devicemapper** is 10GB, which isn't enough
disk space for a large data collection.  The way to work around this is to use the below docker
volume.

    mkdir -p /var/openbmp/kafka
    chmod 777 /var/openbmp/kafka

> The mode of 777 can be changed to chown <user> but you'll have to get that ID 
> by looking at the file owner after starting the container. 


- - -

### 4) Run docker container

#### Environment Variables
Below table lists the environment variables that can be used with ``docker -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
**KAFKA\_FQDN** | hostname | **required** Fully qualified hostname for the docker host/IP of this container, will be used for Kakfa.

#### Run normally

> ##### IMPORTANT
> You must define the **KAFKA_FQDN** as a 'hostname'.  If all containers are running on the same node, this
> hostname can be local specific, such as 'localhost' or 'myhost'. If Kafka is running on a different server,
> than the consumers and producers, then the KAFKA_FQDN should be a valid hostname that can be resolved using DNS.
> This can be internal DNS or manually done by updating the /etc/hosts file on each machine.

    docker run -d \
         --name=openbmp_kafka \
         -e KAFKA_FQDN=localhost \
         -v /var/openbmp/kafka:/data/kafka \
         -p 2181:2181 -p 9092:9092 \
         openbmp/kafka


### Monitoring/Troubleshooting

You can use standard docker exec commands to monitor the log files.  To monitor
kafka, use ```docker exec openbmp_kafka tail -f /var/log/*.log```

Alternatively, it can be easier at times to navigate all the log files from within the container. You can do so using:

    docker exec -it openbmp_kafka bash

You can also monitor one of the topics to see messages.  For example, you can monitor the openbmp.parsed.unicast_prefix topic using:

    docker exec openbmp_kafka /usr/local/kafka/bin/kafka-console-consumer.sh -z localhost --topic openbmp.parsed.unicast_prefix

#### docker logs
You can use ```docker logs openbmp_kafka``` to get the console logs. This is useful if the container exits due to
invalid start or for another reason.

#### System Start/Restart Config (ubuntu 14.04)
By default, the containers will not start automatically on system boot/startup.
You can use the below example to instruct the container to start automatically.

You can read more at [Docker Host Integration](https://docs.docker.com/articles/host_integration/) on how to start containers automatically.

> #### IMPORTANT
> The ```--name=openbmp_kafka``` parameter given to the ```docker run``` command is used with the ```-a openbmp_kafka``` parameter below to start the container by name instead of container ID.  You can use whatever name you want, but make sure to use the same name used in docker run.

    cat <<END > /etc/init/kafka-openbmp.conf
    description "OpenBMP Kafka container"
    author "tim@openbmp.org"
    start on filesystem and started docker
    stop on runlevel [!2345]
    respawn
    script
      /usr/bin/docker start -a openbmp_kafka
    end script
    END
