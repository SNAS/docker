OpenBMP All-In-One Container
----------------------------
All-in-one includes everything needed to run the collector and store the data in MySQL.   You can use this container to test/evaluate OpenBMP as well as run smaller deployments.  Production deployments normally would have distributed collectors and a redundant pair of MySQL/MariaDB servers. 

#### Container Includes
* **Openbmpd** - Latest collector (listening port is TCP 5000)
* **MariaDB 10.0** - MySQL server (listening port TCP 3306)
* **Apache Kafka 0.8.x** - High performing message bus (listening ports are TCP 2181 and 9092)
* **Tomcat/DB_REST** - Latest Rest interface into MySQL/MariaDB (listening port TCP 8001)
* **Openbmp MySQL Consumer** - Latest Consumer that puts all data into MySQL


### Recommended Current Linux Distributions

  1. Ubuntu 14.04/Trusty
  1. CentOS 7/RHEL 7

### 1) Install docker
Docker host should be **Linux x86_64**.   Follow the [Docker Instructions](https://docs.docker.com/installation/) to install docker.  

- - -

### 2) Download the docker image

    docker pull openbmp/aio

- - -
### 3) Create MySQL volumes
MySQL/MariaDB uses a shared container (host) volume so that if you upgrade, restart,
change the container it doesn't loose the DB contents.  **The DB will 
be initialized
if the volume is empty.**  If the volume is not empty, the DB will be left unchanged. 
This can be an issue when the schemas need to change.  Therefore, to reinit the DB and
apply the latest schema use docker run with the ```-e REINIT_DB=1```

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

    mkdir -p /var/openbmp/mysqltmp
    echo "tmpfs /var/openbmp/mysqltmp tmpfs defaults,gid=nogroup,uid=nobody,size=2400M,mode=0777 0 0" >> /etc/fstab
    mount /var/openbmp/mysqltmp

### 4) Run docker container

> #### Memory for MySQL
> Mysql requires a lot of memory in order to run well.   Currently there is not a  consistent way to check on the container memory limit. The ```-e MEM=size_in_GB`` should be specified in gigabytes (e.g. 16 for 16GB of RAM).   If you fail to supply this variable, the default will use **/proc/meminfo** .  In other words, the default is to assume no memory limit. 

#### Environment Variables
Below table lists the environment variables that can be used with ``docker -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
**API\_FQDN** | hostname | **required** Fully qualified hostname for the docker host/IP of this container, will be used for API and Kafka. It is also the collector Admin Id
MEM | RAM in GB | The size of RAM allowed for container in gigabytes. (e.g. ```-e MEM=15```)
OPENBMP_BUFFER | Size in MB | Defines the openbmpd buffer per router for BMP messages. Default is 16 MB.  
REINIT_DB | 1 | If set to 1 the DB will be reinitialized, which is needed to load the new schema sometimes.  This will wipe out the old data and start from scratch.  When this is not set, the old DB is reused.   (e.g. ```-e REINIT_DB=1```)
MYSQL\_ROOT\_PASSWORD | password | MySQL root user password.  The default is **OpenBMP**.  The root password can be changed using [standard MySQL instructions](https://dev.mysql.com/doc/refman/5.6/en/resetting-permissions.html).  If you do change the password, you will need to run the container with this env set.
MYSQL\_OPENBMP\_PASSWORD | password | MySQL openbmp user password.  The default is **openbmp**.  You can change the default openbmp user password using [standard mysql instructions](https://dev.mysql.com/doc/refman/5.6/en/set-password.html).  If you change the openbmp user password you MUST use this env.  

#### Run normally
> ##### IMPORTANT
> Make sure to define **API_FQDN** as a valid hostname or IP address reachable by 
> the client browser.

    docker run -d -e API_FQDN=<hostname or IP> --name=openbmp_aio -e MEM=15 \
         -v /var/openbmp/mysql:/data/mysql \
         -p 3306:3306 -p 2181:2181 -p 9092:9092 -p 5000:5000 -p 8001:8001 \
         openbmp/aio

> ### Allow at least a few minutes for mysql to init the DB on first start



### Monitoring/Troubleshooting
Once the container is running you can run a **HTTP GET http://docker_host:8001/db_rest/v1/routers** to test that the API interface is working. 

You can use standard docker exec commands to monitor the log files.  To monitor 
openbmp, use ```docker exec openbmp_aio tail -f /var/log/openbmpd.log```

Alternatively, it can be easier at times to navigate all the log files from within the container. You can do so using:
    
    docker exec -it openbmp_aio bash

#### System Start/Restart Config (ubuntu 14.04)
By default, the containers will not start automatically on system boot/startup.  You can use the below example to instruct the openbmp/aio container to start automatically. 

You can read more at [Docker Host Integration](https://docs.docker.com/articles/host_integration/) on how to start containers automatically. 

> #### IMPORTANT
> The ```--name=openbmp_aio``` parameter given to the ```docker run``` command is used with the ```-a openbmp_aio``` parameter below to start the container by name instead of container ID.  You can use whatever name you want, but make sure to use the same name used in docker run.

    cat <<END > /etc/init/aio-openbmp.conf
    description "OpenBMP All-In-One container"
    author "tim@openbmp.org"
    start on filesystem and started docker
    stop on runlevel [!2345]
    respawn
    script
      /usr/bin/docker start -a openbmp_aio
    end script
    END
     
     



