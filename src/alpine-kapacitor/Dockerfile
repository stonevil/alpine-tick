FROM quay.io/stonevil/alpine-glibc
MAINTAINER stonevil@gmail.com

ENV KAPACITOR_VERSION=${INFLUXDB_VERSION:-0.13.1}
ENV KAPACITOR_DOWNLOAD_URL=https://dl.influxdata.com/kapacitor/releases/kapacitor-${KAPACITOR_VERSION}_linux_amd64.tar.gz

LABEL com.stonevil.kapacitor.version=${KAPACITOR_VERSION}

RUN ${SOFTWARE_DOWNLOAD_CMD} ${KAPACITOR_DOWNLOAD_URL} | tar -zxvf - -C /tmp && \
  cp -R /tmp/kapacitor-${KAPACITOR_VERSION}*/* / && \
  rm -Rf /tmp/kapacitor-${KAPACITOR_VERSION}* && \
  adduser -D -H -h /data/kapacitor -s /bin/bash kapacitor

ADD _rootfs /
ONBUILD ADD _rootfs_vendor /

EXPOSE 9092

VOLUME ["/data/kapacitor"]

ENV INFLUXDB_SENSORS_DB=${INFLUXDB_SENSORS:-sensors}

ENV INFLUXDB_CONSUMERS_DB=${INFLUXDB_CONSUMERS_DB:-consumers}

ENV INFLUXDB_AGGREGATORS=${INFLUXDB_AGGREGATORS:-aggregators}
ENV INFLUXDB_AGGREGATORS_PASSWORD=${INFLUXDB_AGGREGATORS_PASSWORD:-AggregatorsPassword}

ENV KAPACITOR_HOST=${KAPACITOR_HOST:-localhost}

ENV INFLUXDB_HOST=${INFLUXDB_HOST:-localhost}
ENV INFLUXDB_PORT=${INFLUXDB_PORT:-8086}

ENV SMTP_HOST=${SMTP_HOST:-localhost}
ENV SMTP_PORT=${SMTP_PORT:-25}
ENV SMTP_USER=${SMTP_USER:-}
ENV SMTP_PASSWORD=${SMTP_PASSWORD:-}

CMD mkdir -p /data/kapacitor && chown -Rf kapacitor /data/kapacitor && \
  sed -i -e 's|_KAPACITORHOST_|'"${KAPACITOR_HOST}"'|g' -e 's|_INFLUXDBURL_|'"${INFLUXDB_HOST}:${INFLUXDB_PORT}"'|g' -e 's|_INFLUXDBDBNAME_|'"${INFLUXDB_SENSORS_DB}"'|g' -e 's|_INFLUXDBAGGREGATORS_|'"${INFLUXDB_AGGREGATORS}"'|g' -e 's|_INFLUXDBAGGREGATORSPASSWORD_|'"${INFLUXDB_AGGREGATORS_PASSWORD}"'|g' -e 's|_SMTPHOST_|'"${SMTP_HOST}"'|g' -e 's|_SMTPPORT_|'"${SMTP_PORT}"'|g' -e 's|_SMTPUSER_|'"${SMTP_USER}"'|g' -e 's|_SMTPPASSWORD_|'"${SMTP_PASSWORD}"'|g' /etc/kapacitor/kapacitor.conf && \
  gosu kapacitor kapacitord -config /etc/kapacitor/kapacitor.conf
