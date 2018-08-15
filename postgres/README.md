OpenBMP Postgres Container
----------------------------
This container provides PostgreSQL backend to OpenBMP. It requires the following other containers:

- **openbmp/collector**
- **openbmp/kafka** or some Kafka instance.  Does not have to be the OpenBMP one. 

#### Container Includes the following
- **Postgres** - Latest postgres 10.x release. TCP port 5432
- **TimescaleDB** - Latest version of TimescaleDB
- **RPKI Validator** - RPKI Validator 2.24. TCP port 8080 

### Recommended Current Linux Distributions and system requirements

  1. Ubuntu 18.04/Bionic
  1. CentOS 7/RHEL 7

You will need to dedicate space for the postgres instance.  Normally two partitions are used.  A good
starting size for postgres main is 500GB and postgres ts (timescaleDB) is 1TB.  Both disks
should be fast SSD. ZFS can be used on either of them to add compression. The size you need will depend
on the number of NLRI's and updates per second.

#### NOTE: Postgres can be killed by the Linux OOM-Killer
This is very bad as it causes Postgres to restart. This will happen because postgres uses a large shared buffer,
which causes the OOM to believe it's using a lot of VM.     

It is suggested to run the postgres server with the following Linux settings:

    # Update runtime
    sysctl -w vm.vfs_cache_pressure=500
    sysctl -w vm.swappiness=10
    sysctl -w vm.min_free_kbytes=1000000
    sysctl -w vm.overcommit_memory=2
    sysctl -w vm.overcommit_ratio=95   

    # Update startup    
    echo "vm.vfs_cache_pressure=500" >> /etc/sysctl.conf
    echo "vm.min_free_kbytes=1000000" >> /etc/sysctl.conf
    echo "vm.swappiness=10" >> /etc/sysctl.conf
    echo "vm.overcommit_memory=2" >> /etc/sysctl.conf
    echo "vm.overcommit_ratio=95" >> /etc/sysctl.conf


See Postgres [hugepages](https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES) for
details on how to enable and use hugepages.   Some Linux distributions enable **transparent hugepages** which
will prevent the ability to configure ```vm.nr_hugepages```. If you find that you cannot set ```vm.nr_hugepages```,
then try the below:

    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
    sync && echo 3 > /proc/sys/vm/drop_caches


### 1) Install docker
Follow the [Docker Instructions](https://docs.docker.com/install) to install docker CE.  

- - -

### 2) Add persistent volumes

Persistent volumes make it possible for upgrades with out loosing any data. 

#### (a) Create persistent config location

    mkdir -p /var/openbmp/config
    chmod 777 /var/openbmp/config

##### config/hosts
You can add custom host entries so that the collector will reverse lookup IP addresses
using a persistent hosts file.

Run docker with ```-v /var/openbmp/config:/config``` to make use of the persistent config files.

##### config/obmp-psql.yml
If the [obmp-psql.yml](https://github.com/OpenBMP/obmp-postgres/blob/master/src/main/resources/obmp-psql.yml) file
does not exist, a default one will be created. You should update this based on your settings. This file
is inline documented.  

#### (b) Create persistent postgres locations

*You should use fast SSD and/or ZFS.*  Size of these locations/mount points are directly related to the 
number of NLRI's maintained and number of changes/updates per second. 

> TODO: Will post numbers of how to determine the disk size needed.  For now, if you have less
> than 50,000,00 prefixes, then you can use 1TB.  If you have more than that, you should consider
> multiple disks.  ZFS can make your life easier as you can easily add disks and it supports compression.    

- **postgres/main** - This location will be used for the main postgres data
files and tables. 

> This really should be a mount point to a dedicated filesystem

```
    mkdir -p /var/openbmp/postgres/main
    chmod 7777 /var/openbmp/postgres/main
``` 

- **postgres/ts** - This location will be used for the time series postgres tables

> This really should be a mount point to a dedicated filesystem

```
    mkdir -p /var/openbmp/postgres/ts
    chmod 7777 /var/openbmp/postgres/ts
```

### 3) Run docker container

> Running the docker container for the first time will download the container image. 

#### Environment Variables
Below table lists the environment variables that can be used with ``docker run -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
KAFKA\_FQDN | hostanme or IP | Kafka broker hostname.  Hostname can be an IP address.
ENABLE_RPKI | 1 | Set to 1 to eanble RPKI. RPKI is disabled by default
ENABLE_IRR | 1 | Set to 1 to enable IRR. IRR is disabled by default
MEM | number | Number value in GB to allocate to Postgres.  This will be the shared_buffers value.
PGUSER | username | Postgres username, default is **openbmp**
PGPASSWORD | password | Postgres password, default is **openbmp**
PGDATABASE | database | Name of postgres database, default is **openbmp**


#### Run normally

> ##### NOTE:
> If the container fails to start, it's likely due to the configuration. Check using
> ```docker logs openbmp_psql```

```
docker run -d --name openbmp_psql \
	-h obmp-psql \
	-e KAFKA_FQDN=kafka.domain \
	--add-host kafka.domain:172.17.0.1 \
	-e MEM=16 \
	-v /var/openbmp/config:/config \
	-v /var/openbmp/postgres/main:/data/main \
	-v /var/openbmp/postgres/ts:/data/ts \
	-p 5432:5432 -p 9005:9005 -p 8080:8080 \
	openbmp/postgres
```

> The above uses **kafka.domain** and maps that in the container to the **docker0** IP. This 
> ensures that the kafka hostname will resolve correctly within the container for the
> openbmp/kafka instance that runs on the same host. Change this to your Kafka bootstrap
> server.  Make sure that all broker hostnames can be resolved by the container. When
> in doubt, use **--add-host** or add entries to the **/var/openbmp/config/hosts** file. 

### Monitoring/Troubleshooting

Useful commands:

- docker logs openbmp_psql
- docker exec openbmp_psql tail -f /var/log/obmp-psql.log
- docker exec openbmp_psql tail -f /var/log/postgresql/postgresql-10-main.log 
- docker exec -it openbmp_psql bash

