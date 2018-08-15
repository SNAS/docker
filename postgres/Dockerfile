# Postgres Backend: openbmp/postgres
#
#  Copyright (c) 2018 Cisco Systems, Inc. and others.  All rights reserved.
#  Copyright (c) 2018 Tim Evens (tim@evensweb.com).  All rights reserved.
#
#  This program and the accompanying materials are made available under the
#  terms of the Eclipse Public License v1.0 which accompanies this distribution,
#  and is available at http://www.eclipse.org/legal/epl-v10.html
#
# Author: Tim Evens <tim@evensweb.com>
#
# BUILD: docker build -t openbmp/postgres .

# Pull base image.
FROM debian:stretch-slim

# Add files.
ADD scripts/install /tmp/
ADD scripts/run /usr/sbin/

ARG BUILD_NUMBER=0

# Proxy servers
#ENV http_proxy http://proxy:80
#ENV https_proxy http://proxy:80
#ENV no_proxy "domain.com"

#----------------------------------
# Run Install script
RUN /tmp/install

#----------------------------------
# Define mount points.
VOLUME ["/config"]

# Postgres main disk
VOLUME ["/data/main"]

# Postgres time series disk
VOLUME ["/data/ts"]

#----------------------------------
# Define working directory.
WORKDIR /tmp

# Define default command.
CMD ["/usr/sbin/run"]

#----------------------------------
# Expose ports.

# Postgres
EXPOSE 5432

# Consumer JMX console
EXPOSE 9005

# RPKI Validator port
EXPOSE 8080
