FROM armhf/debian:stretch-slim
MAINTAINER Dennis Uhlemann <dennis.uhlemann@agenterp.com>

COPY . /container
RUN /container/build.sh

ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8"

ENTRYPOINT ["/container/tool/run"]
