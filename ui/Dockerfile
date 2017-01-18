# Copyright (c) 2015-2016 Cisco Systems, Inc. and others.  All rights reserved.
#
# OpenBMP/UI Dockerfile
#

# Pull base image.
FROM alpine:3.3

ENV BUILD_NUMBER ${BUILD_NUMBER}

# Add files.
ADD scripts/install /tmp/
ADD scripts/run /usr/sbin/
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
