# Copyright (c) 2015-2016 Cisco Systems, Inc. and others.  All rights reserved.
#
# OpenBMP/UI Dockerfile
#

# Pull base image.
FROM alpine:3.6

ARG BUILD_NUMBER=0

# Add files.
ADD scripts/install /tmp/
ADD scripts/run /usr/sbin/
ADD files/nginx.conf /etc/nginx/
COPY files/bmpUI.war /tmp/

# Run Install script
RUN apk add --update bash && /tmp/install

# Define environment variables.

# Define mount points.
VOLUME ["/config"]

# Define working directory.
WORKDIR /tmp

# Define default command.
ENTRYPOINT ["/usr/sbin/run"]

# UI
EXPOSE 8000
