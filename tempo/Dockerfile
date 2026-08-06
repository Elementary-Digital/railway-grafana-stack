# Pin to Tempo 2.7.x — `latest` (Tempo 3) removed top-level ingester/compactor
# config keys and broke the previous Railway deployment.
ARG VERSION=2.7.2

FROM grafana/tempo:${VERSION}

COPY tempo.yml /etc/tempo/tempo.yml

CMD ["-config.file=/etc/tempo/tempo.yml"]
