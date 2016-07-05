FROM quay.io/stonevil/alpine-glibc
MAINTAINER stonevil@gmail.com

ENV INFLUXDB_VERSION=${INFLUXDB_VERSION:-0.13.0}
ENV INFLUXDB_DOWNLOAD_URL=https://dl.influxdata.com/influxdb/releases/influxdb-${INFLUXDB_VERSION}_linux_amd64.tar.gz
ENV INFLUXDB_BASE_PATH=/opt
ENV INFLUXDB_PATH $INFLUXDB_BASE_PATH/influxdb
ENV PATH $INFLUXDB_PATH/usr/bin:$PATH

LABEL com.stonevil.influxdb.version=${INFLUXDB_VERSION}

RUN ${SOFTWARE_DOWNLOAD_CMD} ${INFLUXDB_DOWNLOAD_URL} | tar -zxvf - -C /tmp && \
  cp -Rf /tmp/influxdb*/* / && rm -Rf /tmp/influxdb* && \
  adduser -D -H -h /data/influxdb -s /bin/bash influx && \
  ${APK_CACHE_RM}

ADD _rootfs /
ONBUILD ADD _rootfs_vendor /

EXPOSE 8083 8086 25826

VOLUME ["/data/influxdb"]

ENV INFLUXDB_INIT=${INFLUXDB_INIT:-true}

ENV INFLUXDB_ADMIN=${INFLUXDB_ADMIN:-admin}
ENV INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_ADMIN_PASSWORD:-AdminPassword}

ENV INFLUXDB_SENSORS_DB=${INFLUXDB_SENSORS_DB:-sensors}
ENV INFLUXDB_SENSORS=${INFLUXDB_SENSORS:-sensors}
ENV INFLUXDB_SENSORS_PASSWORD=${INFLUXDB_SENSORS_PASSWORD:-SensorsPassword}

ENV INFLUXDB_CONSUMERS_DB=${INFLUXDB_CONSUMERS_DB:-consumers}
ENV INFLUXDB_CONSUMERS=${INFLUXDB_CONSUMERS:-consumers}
ENV INFLUXDB_CONSUMERS_PASSWORD=${INFLUXDB_CONSUMERS_PASSWORD:-ConsumersPassword}

ENV INFLUXDB_COLLECTD_DB=${INFLUXDB_COLLECTD_DB:-collectd}

ENV INFLUXDB_AGGREGATORS=${INFLUXDB_AGGREGATORS:-aggregators}
ENV INFLUXDB_AGGREGATORS_PASSWORD=${INFLUXDB_AGGREGATORS_PASSWORD:-AggregatorsPassword}

CMD mkdir -p /data/influxdb && chown -Rf influx /data/influxdb && \
  (if ${INFLUXDB_INIT}; then (until [ `curl -L -s -w "%{http_code}\\n" "127.0.0.1:8086" -o /dev/null` -eq 404 ]; do echo "Waiting for InfluxDB to initialise DB" && sleep 5; done && \
  (echo "Initilasing users and databases"; \
  influx -unsafeSsl -execute "CREATE USER ${INFLUXDB_ADMIN} WITH PASSWORD '${INFLUXDB_ADMIN_PASSWORD}' WITH ALL PRIVILEGES" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "CREATE DATABASE ${INFLUXDB_SENSORS_DB}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "CREATE USER ${INFLUXDB_SENSORS} WITH PASSWORD '${INFLUXDB_SENSORS_PASSWORD}'" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "GRANT ALL ON ${INFLUXDB_SENSORS_DB} TO ${INFLUXDB_SENSORS}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "CREATE DATABASE ${INFLUXDB_CONSUMERS_DB}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "CREATE USER ${INFLUXDB_CONSUMERS} WITH PASSWORD '${INFLUXDB_CONSUMERS_PASSWORD}'" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "GRANT READ ON ${INFLUXDB_CONSUMERS_DB} TO ${INFLUXDB_CONSUMERS}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "CREATE USER ${INFLUXDB_AGGREGATORS} WITH PASSWORD '${INFLUXDB_AGGREGATORS_PASSWORD}'" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "GRANT ALL ON ${INFLUXDB_SENSORS_DB} TO ${INFLUXDB_AGGREGATORS}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "GRANT ALL ON ${INFLUXDB_CONSUMERS_DB} TO ${INFLUXDB_AGGREGATORS}" && \
  influx -unsafeSsl -username ${INFLUXDB_ADMIN} -password ${INFLUXDB_ADMIN_PASSWORD} -execute "GRANT ALL ON ${INFLUXDB_COLLECTD_DB} TO ${INFLUXDB_AGGREGATORS}" ); ); fi) & \
  gosu influx influxd -config /etc/influxdb/influxdb.conf
