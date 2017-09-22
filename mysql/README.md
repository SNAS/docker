OpenBMP MySQL Container
-----------------------
Mysql includes MariaDB,  mysql consumer and the rest API.  This container providers everything needed
to get started with collecting data from the OpenBMP Kafka integration.

#### Container Includes
* **MariaDB 10.0** - MySQL server (listening port TCP 3306)
* **Openbmp MySQL Consumer** - Latest Consumer that puts all data into MySQL
* **DB_REST** - Latest Rest interface for the DB


### Recommended Current Linux Distributions

  1. Ubuntu 14.04/Trusty
  1. CentOS 7/RHEL 7

### 1) Install docker
Docker host should be **Linux x86_64**.   Follow the [Docker Instructions](https://docs.docker.com/installation/) to install docker.  

- - -

### 2) Download the docker image

    docker pull openbmp/mysql

- - -
### 3) Create MySQL volumes
MySQL/MariaDB uses a shared container (host) volume so that if you upgrade, restart, change the container it doesn't loose the DB contents.  **The DB will be initialized if the volume is empty.**  If the volume is not empty, the DB will be left unchanged.  This can be an issue when the schemas need to change. Therefore, to reinit the DB and apply the latest schema use docker run with the ```-e REINIT_DB=1```

When starting the container you will need to map a host file system to **/data/mysql** for the container.  You do this using the ```-v <host path>:/data/mysql```.  The below examples default to the host path of ```/var/openbmp/mysql```

#### (Optional) MySQL Temporary Table Space
Large queries or queries that involve sorting/counting/... will use a temporary table on disk.   It is recommended that
you use a **tmpfs** memory mount point for this.  Docker will not allow the container to mount tmpfs without having
CAP\_SYS\_ADMIN capability (``-privileged``).  To work around this limitation, the container will not create the tmpfs.  Instead
the container will use ``/var/mysqltmp`` which can be a volume to the host system tmpfs mount point.   The host system 
should create a tmpfs and then map that as a volume in docker using ``-v /var/openbmp/mysqltmp:/var/mysqltmp``

#### On host create mysql shared dir
    mkdir -p /var/openbmp/mysql
    chmod 777 /var/openbmp/mysql 

> The mode of 777 can be changed to chown <user> but you'll have to get that ID 
> by looking at the file owner after starting the container. 
    

#### On host create tmpfs (as root)
> ONLY USE TMPFS IF YOU HAVE ENOUGH MEMORY

    mkdir -p /var/openbmp/mysqltmp
    echo "tmpfs /var/openbmp/mysqltmp tmpfs defaults,gid=nogroup,uid=nobody,size=2400M,mode=0777 0 0" >> /etc/fstab
    mount /var/openbmp/mysqltmp


### 4) [OPTIONAL] Add persistent configs

#### On host create persistent config location
    mkdir -p /var/openbmp/config
    chmod 777 /var/openbmp/config

#### config/hosts
You can add custom host entries so that the collector will reverse lookup IP addresses
using a persistent hosts file.

### 5) Run docker container

> #### Memory for MySQL
> Mysql requires a lot of memory in order to run well.   Currently there is not a consistent way to check on the container memory limit. The ```-e MEM=size_in_GB`` should be specified in gigabytes (e.g. 16 for 16GB of RAM).   If you fail to supply this variable, the default will use **/proc/meminfo** .  In other words, the default is to assume no memory limit.

#### Environment Variables
Below table lists the environment variables that can be used with ``docker -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
MEM | RAM in GB | The size of RAM allowed for container in gigabytes. (e.g. ```-e MEM=15```)
GROUP\_ID | string | The Kafka consumer group ID, default is 'openbmp-mysql-consumer'
KAFKA\_FQDN | hostanme or IP | Kafka broker hostname[:port].  Hostname can be an IP address
REINIT_DB | 1 | If set to 1 the DB will be reinitialized, which is needed to load the new schema sometimes.  This will wipe out the old data and start from scratch.  When this is not set, the old DB is reused.   (e.g. ```-e REINIT_DB=1```)
MYSQL\_ROOT\_PASSWORD | password | MySQL root user password.  The default is **OpenBMP**.  The root password can be changed using [standard MySQL instructions](https://dev.mysql.com/doc/refman/5.6/en/resetting-permissions.html).  If you do change the password, you will need to run the container with this env set.
MYSQL\_OPENBMP\_PASSWORD | password | MySQL openbmp user password.  The default is **openbmp**.  You can change the default openbmp user password using [standard mysql instructions](https://dev.mysql.com/doc/refman/5.6/en/set-password.html).  If you change the openbmp user password you MUST use this env.
NUM\_MYSQL\_CONSUMERS | int | Number of consumers to run. This should match the number of Kafka partitions, default is 6.   

#### Run normally

> ##### IMPORTANT
> You must define the **KAFKA_FQDN** as a 'hostname'.  If all containers are running on the same node, this
> hostname can be local specific, such as 'localhost' or 'myhost'. If Kafka is running on a different server,
> than the consumers and producers, then the KAFKA_FQDN should be a valid hostname that can be resolved using DNS.
> This can be internal DNS or manually done by updating the /etc/hosts file on each machine.

    docker run -d --name=openbmp_mysql -e KAFKA_FQDN=localhost -e MEM=15 \
         -v /var/openbmp/mysql:/data/mysql -v /var/openbmp/config:/config \
         -p 3306:3306 -p 8001:8001 \
         openbmp/mysql

> ### Allow at least a few minutes for mysql to init the DB on first start


### Monitoring/Troubleshooting
Once the container is running you can connect to MySQL using any ODBC/JDBC/Mysql client.

You can also connect to the DB REST interface on port 8001.  For example: http://localhost:8001/db_rest/v1/routers

You can use standard docker exec commands to monitor the log files.  To monitor 
openbmp, use ```docker exec openbmp_mysql tail -f /var/log/*.log```

Alternatively, it can be easier at times to navigate all the log files from within the container. You can do so using:
    
    docker exec -it openbmp_mysql bash

#### docker logs
You can use ```docker logs openbmp_mysql``` to get the console logs. This is useful if the container exits due to
invalid start or for another reason.

#### System Start/Restart Config (ubuntu 14.04)
By default, the containers will not start automatically on system boot/startup.  You can use the below example to instruct the container to start automatically.

You can read more at [Docker Host Integration](https://docs.docker.com/articles/host_integration/) on how to start containers automatically. 

> #### IMPORTANT
> The ```--name=openbmp_mysql``` parameter given to the ```docker run``` command is used with the ```-a openbmp_mysql``` parameter below to start the container by name instead of container ID.  You can use whatever name you want, but make sure to use the same name used in docker run.

    cat <<END > /etc/init/mysql-openbmp.conf
    description "OpenBMP MySQL container"
    author "tim@openbmp.org"
    start on filesystem and started docker
    stop on runlevel [!2345]
    respawn
    script
      /usr/bin/docker start -a openbmp_mysql
    end script
    END
     
     



