ARG VERSION=3.4

FROM grafana/loki:${VERSION}

# Copy the loki.yml configuration file to the container
COPY loki.yml /etc/loki/loki-confi.yaml

# Railway mounts the persistent volume at /loki owned by root, but the upstream
# grafana/loki image runs as uid 10001, so it cannot create tsdb-cache/tsdb-index
# and crash-loops with "permission denied". Running as root lets Loki write the
# volume while keeping log persistence. Traffic is internal-only (no public port).
USER root

# Command to run Loki
CMD ["-config.file=/etc/loki/loki-confi.yaml"]