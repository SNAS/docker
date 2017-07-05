# All-In-One openbmp/aio
#
#  Copyright (c) 2013-2017 Cisco Systems, Inc. and others.  All rights reserved.
#
#  This program and the accompanying materials are made available under the
#  terms of the Eclipse Public License v1.0 which accompanies this distribution,
#  and is available at http://www.eclipse.org/legal/epl-v10.html
#
# Author: Tim Evens <tim@openbmp.org>
#
# BUILD: docker build -t openbmp/aio .
# SAVE: docker save -o openbmp-aio-$(date +"%m%d%y").docker_img.tar openbmp/aio:latest
#       gzip openbmp-aio-$(date +"%m%d%y").docker_img.tar

# Pull base image.
FROM ubuntu:xenial

# Proxy/if behind firewall
#   Update your /etc/sysconfig/docker or /etc/default/docker file by adding proxy envs there using export <var>=<val>

# Add files.
ADD defaults/my.cnf /etc/mysql/my.cnf.tmpl
ADD defaults/tomcat8 /tmp/
ADD scripts/install /tmp/
ADD scripts/run_all_in_one /usr/sbin/
ADD files/geo_ip.db /usr/local/
ADD files/users.db /usr/local/users.db
COPY files/whois.db.gz /usr/local/

ARG BUILD_NUMBER=0

# Proxy servers
#ENV http_proxy http://proxy:80
#ENV https_proxy http://proxy:80
#ENV no_proxy "domain.com"

# Run Install script
RUN /tmp/install

# Define mount points.
VOLUME ["/data/mysql"]
VOLUME ["/var/mysqltmp"]
VOLUME ["/config"]

# Define working directory.
WORKDIR /tmp

# Define default command.
CMD ["/usr/sbin/run_all_in_one"]

# Expose ports.
# openbmpd/collector
EXPOSE 5000         

# MySQL/MariaDB
EXPOSE 3306         

# Zookeeper
EXPOSE 2181

# Kafka
EXPOSE 9092
EXPOSE 9999

# REST/API
EXPOSE 8001         
