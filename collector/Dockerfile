# Collector: openbmp/collector
#
#  Copyright (c) 2013-2015 Cisco Systems, Inc. and others.  All rights reserved.
#
#  This program and the accompanying materials are made available under the
#  terms of the Eclipse Public License v1.0 which accompanies this distribution,
#  and is available at http://www.eclipse.org/legal/epl-v10.html
#
# Author: Tim Evens <tim@openbmp.org>
#
# BUILD: docker build -t openbmp/collector .

# Pull base image.
FROM ubuntu:trusty

# Proxy/if behind firewall
#   Update your /etc/sysconfig/docker or /etc/default/docker file by adding proxy envs there using export <var>=<val>

# Add files.
ADD scripts/install /tmp/
ADD scripts/run /usr/sbin/

ARG BUILD_NUMBER=0

# Proxy servers
#ENV http_proxy http://proxy:80
#ENV https_proxy http://proxy:80
#ENV no_proxy "domain.com"

# Run Install script
RUN /tmp/install

# Define mount points.
VOLUME ["/config"]

# Define working directory.
WORKDIR /tmp

# Define default command.
CMD ["/usr/sbin/run"]

# Expose ports.
# openbmpd/collector
EXPOSE 5000         
