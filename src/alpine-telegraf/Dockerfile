FROM quay.io/stonevil/alpine-glibc
MAINTAINER stonevil@gmail.com

ENV TELEGRAF_VERSION=${TELEGRAF_VERSION:-0.13.1}
ENV TELEGRAF_DOWNLOAD_URL=https://dl.influxdata.com/telegraf/releases/telegraf-${TELEGRAF_VERSION}_linux_amd64.tar.gz

LABEL com.stonevil.telegraf.version=${TELEGRAF_VERSION}

RUN ${SOFTWARE_DOWNLOAD_CMD} ${TELEGRAF_DOWNLOAD_URL} | tar -zxvf - -C /tmp && \
  cp -Rf /tmp/telegraf*/* / && rm -Rf /tmp/telegraf* && \
  adduser -D -H -h /data/telegraf -s /bin/bash telegraf && \
  ${APK_CACHE_RM}

ADD _rootfs /
ONBUILD ADD _rootfs_vendor /

EXPOSE 8125/udp 8092/udp 8094

ENV INFLUXDB_SENSORS_DB=${INFLUXDB_SENSORS_DB:-sensors}
ENV INFLUXDB_SENSORS=${INFLUXDB_SENSORS:-sensors}
ENV INFLUXDB_SENSORS_PASSWORD=${INFLUXDB_SENSORS_PASSWORD:-SensorsPassword}

ENV INFLUXDB_HOST=${INFLUXDB_HOST:-localhost}
ENV INFLUXDB_PORT=${INFLUXDB_PORT:-8086}

ENV DOCKER_ENDPOINT=${DOCKER_ENDPOINT:-unix:///var/run/docker.sock}

CMD mkdir -p /data/telegraf && chown -Rf telegraf /data/telegraf && \
  sed -i -e 's|_INFLUXDBURL_|'"${INFLUXDB_HOST}:${INFLUXDB_PORT}"'|g' -e 's|_INFLUXDBSENSORSDB_|'"${INFLUXDB_SENSORS_DB}"'|g' -e 's|_INFLUXDBSENSORS_|'"${INFLUXDB_SENSORS}"'|g' -e 's|_INFLUXDBSENSORSPASSWORD_|'"${INFLUXDB_SENSORS_PASSWORD}"'|g' -e 's|_DOCKERENDPOINT_|'"${DOCKER_ENDPOINT}"'|g' /etc/telegraf/telegraf.conf && \
  telegraf -config /etc/telegraf/telegraf.conf
