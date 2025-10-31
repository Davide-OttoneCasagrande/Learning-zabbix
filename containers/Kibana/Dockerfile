# Use the official Kibana base image
FROM docker.elastic.co/kibana/kibana:8.17.1

ENV ELASTICSEARCH_HOSTS=http://elasticsearch:9200

USER root

# Install Zabbix agent for Ubuntu 20.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends wget gnupg2 procps ca-certificates && \
    wget -q https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest+ubuntu20.04_all.deb -O /tmp/zabbix-release.deb && \
    dpkg -i /tmp/zabbix-release.deb && \
    apt-get update && \
    apt-get install -y --no-install-recommends zabbix-agent2 && \
    rm -rf /var/lib/apt/lists/* /tmp/zabbix-release.deb

# Configure Zabbix agent
RUN sed -i 's/Server=127.0.0.1/Server=172.28.0.10/g' /etc/zabbix/zabbix_agent2.conf && \
    sed -i 's/ServerActive=127.0.0.1/ServerActive=172.28.0.10/g' /etc/zabbix/zabbix_agent2.conf && \
    sed -i 's/Hostname=Zabbix server/Hostname=kibana_01/g' /etc/zabbix/zabbix_agent2.conf && \
    echo "ListenIP=0.0.0.0" >> /etc/zabbix/zabbix_agent2.conf

# Create directories
RUN mkdir -p /var/log/zabbix /var/run/zabbix && \
    chown -R zabbix:zabbix /var/log/zabbix /var/run/zabbix

# Startup script
COPY --chmod=755 <<'EOF' /start.sh
#!/bin/bash
set -e
echo "Starting Zabbix agent..."
/usr/sbin/zabbix_agent2 -c /etc/zabbix/zabbix_agent2.conf &
sleep 3
if pgrep zabbix_agent2 > /dev/null; then
    echo "✓ Zabbix agent running (PID: $(pgrep zabbix_agent2))"
else
    echo "✗ WARNING: Zabbix agent not detected"
fi
echo "Starting Kibana..."
exec su -s /bin/bash kibana -c "/usr/share/kibana/bin/kibana"
EOF

EXPOSE 5601 10050
USER root
CMD ["/bin/bash", "/start.sh"]