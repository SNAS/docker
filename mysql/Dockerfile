# MySQL: openbmp/mysql
#
#  Copyright (c) 2013-2015 Cisco Systems, Inc. and others.  All rights reserved.
#
#  This program and the accompanying materials are made available under the
#  terms of the Eclipse Public License v1.0 which accompanies this distribution,
#  and is available at http://www.eclipse.org/legal/epl-v10.html
#
# Author: Tim Evens <tim@openbmp.org>
#
# BUILD: docker build -t openbmp/mysql .

# Pull base image.
FROM ubuntu:xenial

ARG BUILD_NUMBER=0

# Proxy/if behind firewall
#   Update your /etc/sysconfig/docker or /etc/default/docker file by adding proxy envs there using export <var>=<val>

# Add files.
ADD files/my.cnf /etc/mysql/my.cnf.tmpl
ADD files/tomcat8 /tmp/
ADD files/geo_ip.db /usr/local/
COPY files/whois.db.gz /usr/local/
ADD files/users.db /usr/local/users.db
ADD scripts/install /tmp/
ADD scripts/run /usr/sbin/

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
CMD ["/usr/sbin/run"]

# Expose ports.
# MySQL/MariaDB
EXPOSE 3306         

# rest API
EXPOSE 8001
