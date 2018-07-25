FROM armhf/debian:stretch-slim
MAINTAINER Dennis Uhlemann <dennis.uhlemann@agenterp.com>

COPY ./qemu-arm-static /usr/bin/qemu-arm-static
COPY . /container
RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "-c", "/bin/echo Hello from ARM container" ]
RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "/container/build.sh"]

ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8"

ENTRYPOINT ["/container/tool/run"]
