# Pin to Tempo 2.7.x — `latest` (Tempo 3) removed top-level ingester/compactor
# config keys and broke the previous Railway deployment.
ARG VERSION=2.7.2

FROM grafana/tempo:${VERSION}

# Railway mounts the persistent volume at /var/tempo owned by root, but the
# upstream grafana/tempo image runs as uid 10001, so it cannot create traces/wal
# and crash-loops with "mkdir /var/tempo/traces: permission denied". Running as
# root lets Tempo write the volume. Traffic is internal-only (no public port).
USER root

COPY tempo.yml /etc/tempo/tempo.yml

CMD ["-config.file=/etc/tempo/tempo.yml"]
