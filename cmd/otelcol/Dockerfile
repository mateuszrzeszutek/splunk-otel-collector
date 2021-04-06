FROM alpine:3.12
ARG SMART_AGENT_RELEASE_URL
ARG BASE_DIR="/usr/lib/splunk-otel-collector"

ENV SPLUNK_BUNDLE_DIR=$BASE_DIR/agent-bundle
ENV SPLUNK_COLLECTD_DIR=$SPLUNK_BUNDLE_DIR/run/collectd
ENV SPLUNK_CONFIG=/etc/otel/collector/agent_config.yaml

RUN apk --update add ca-certificates

COPY otelcol /
# Note that this shouldn't be necessary, but in some cases the file seems to be copied with the execute bit lost (see #1317)
RUN chmod 755 /otelcol

RUN  wget -O signalfx-agent.tar.gz "$SMART_AGENT_RELEASE_URL" && \
     tar -xzf signalfx-agent.tar.gz && \
     mkdir -p "$BASE_DIR" && \
     mv signalfx-agent "$SPLUNK_BUNDLE_DIR" && \
     $SPLUNK_BUNDLE_DIR/bin/patch-interpreter $SPLUNK_BUNDLE_DIR && \
     rm -f "$SPLUNK_BUNDLE_DIR/bin/signalfx-agent" \
           "$SPLUNK_BUNDLE_DIR/bin/agent-status" \
           signalfx-agent.tar.gz

COPY config/collector/gateway_config.yaml    /etc/otel/collector/gateway_config.yaml
COPY config/collector/otlp_config_linux.yaml /etc/otel/collector/otlp_config_linux.yaml
COPY config/collector/agent_config.yaml      /etc/otel/collector/agent_config.yaml

ENTRYPOINT ["/otelcol"]

EXPOSE 13133 14250 14268 55680 4317 6060 7276 8888 9411 9443
