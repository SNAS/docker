OpenBMP All-In-One Container
----------------------------
All-in-one includes everything needed to run the collector and store the data in MySQL.   You can use this container to test/evaluate OpenBMP as well as run smaller deployments.  Production deployments normally would have distributed collectors and a redundant pair of MySQL/MariaDB servers. 

#### Container Includes
* **Openbmpd** - Latest collector (listening port is TCP 5000)
* **MariaDB 10.0** - MySQL server (listening port TCP 3306)
* **Apache Kafka 0.9.0.1** - High performing message bus (listening ports are TCP 2181 and 9092)
* **Tomcat/DB_REST** - Latest Rest interface into MySQL/MariaDB (listening port TCP 8001)
* **Openbmp MySQL Consumer** - Latest Consumer that puts all data into MySQL
* **RPKI Validator 2.21** - RPKI Validator - see https://github.com/RIPE-NCC/rpki-validator


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
MySQL/MariaDB uses a shared container (host) volume so that if you upgrade, restart, change the container it doesn't loose the DB contents.  **The DB will be initialized if the volume is empty.**  If the volume is not empty, the DB will be left unchanged.  This can be an issue when the schemas need to change. Therefore, to reinit the DB and apply the latest schema use docker run with the ```-e REINIT_DB=1```

When starting the container you will need to map a host file system to **/data/mysql** for the container.  You do this using the ```-v <host path>:/data/mysql```.  The below examples default to the host path of ```/var/openbmp/mysql```

#### On host create mysql shared dir
    mkdir -p /var/openbmp/mysql
    chmod 777 /var/openbmp/mysql 

> The mode of 777 can be changed to chown <user> but you'll have to get that ID 
> by looking at the file owner after starting the container. 
    

### 4) [OPTIONAL] Add persistent configs

#### On host create persistent config location

    mkdir -p /var/openbmp/config
    chmod 777 /var/openbmp/config

#### config/hosts
You can add custom host entries so that the collector will reverse lookup IP addresses
using a persistent hosts file.

Run docker with ```-v /var/openbmp/config:/config``` to make use of the persistent config files. 

#### config/openbmpd.conf
You can provide a customized **openbmpd.conf**.  See [Config Example](https://github.com/OpenBMP/openbmp/blob/master/Server/openbmpd.conf)

#### config/rpki/tal/*
You can place/update the RPKI TAL's by placing them in the config/rpki/tal directory.

> ##### ARIN is not included. You must add that TAL yourself.
> To access ARIN's TAL, you will have to agree to ARIN's Relying Party Agreement. After
> that, the TAL will be emailed to the recipient. Please visit this ARIN web page for
> more information: http://www.arin.net/public/rpki/tal/index.xhtml


### 5) Run docker container

> #### Memory for MySQL
> Mysql requires a lot of memory in order to run well.   Currently there is not a  consistent way to check on the container memory limit. The ```-e MEM=size_in_GB`` should be specified in gigabytes (e.g. 16 for 16GB of RAM).   If you fail to supply this variable, the default will use **/proc/meminfo** .  In other words, the default is to assume no memory limit. 

#### Environment Variables
Below table lists the environment variables that can be used with ``docker -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
**API\_FQDN** | hostname | **required** Fully qualified hostname for the docker host/IP of this container, will be used for API and Kafka.  You can use **localhost** if there are no external consumers.
**ADMIN\_ID** | string | The collector's admin ID.  This defaults to **collector**, but can be any string to identify this collector instance.
MEM | RAM in GB | The size of RAM allowed for container in gigabytes. (e.g. ```-e MEM=15```)
REINIT_DB | 1 | If set to 1 the DB will be reinitialized, which is needed to load the new schema sometimes.  This will wipe out the old data and start from scratch.  When this is not set, the old DB is reused.   (e.g. ```-e REINIT_DB=1```)
MYSQL\_ROOT\_PASSWORD | password | MySQL root user password.  The default is **OpenBMP**.  The root password can be changed using [standard MySQL instructions](https://dev.mysql.com/doc/refman/5.6/en/resetting-permissions.html).  If you do change the password, you will need to run the container with this env set.
MYSQL\_OPENBMP\_PASSWORD | password | MySQL openbmp user password.  The default is **openbmp**.  You can change the default openbmp user password using [standard mysql instructions](https://dev.mysql.com/doc/refman/5.6/en/set-password.html).  If you change the openbmp user password you MUST use this env.  

#### Run normally
> ##### IMPORTANT
> Make sure to define **API_FQDN** as a valid hostname or IP address reachable for
> external/remote consumers.  You can use **localhost** if there are no remote
> consumers.

    docker run -d --name=openbmp_aio \
         -e API_FQDN=localhost \
         -v /var/openbmp/mysql:/data/mysql \
         -p 3306:3306 -p 2181:2181 -p 9092:9092 -p 5000:5000 -p 8001:8001 \
         openbmp/aio

> ### Allow at least a few minutes for mysql to init the DB on first start

#### After Install
Once the container is running you can run a **HTTP GET http://docker_host:8001/db_rest/v1/routers** to test that the API interface is working.

> Username: **openbmp**
> Password: **CiscoRA**
> 

The default username/password can be changed in the DB via the **users** table.  The UI can also be used to add/remove and change users.

    INSERT users (username,password,type) values ('tim', PASSWORD('mypassword'), 'admin');


### Monitoring/Troubleshooting

You can use standard docker exec commands to monitor the log files.  To monitor 
openbmp, use ```docker exec openbmp_aio tail -f /var/log/openbmpd.log```

Alternatively, it can be easier at times to navigate all the log files from within the container. You can do so using:
    
    docker exec -it openbmp_aio bash

#### docker logs
You can use ```docker logs openbmp_aio``` to get the console logs. This is useful if the container exits due to
invalid start or for another reason.

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
     
     



