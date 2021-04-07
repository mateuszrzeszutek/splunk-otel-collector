ARG ALPINE_VERSION=3.12

FROM alpine:${ALPINE_VERSION} as certs
RUN apk --update add ca-certificates

FROM alpine:${ALPINE_VERSION} AS otelcol
COPY otelcol /
# Note that this shouldn't be necessary, but in some cases the file seems to be
# copied with the execute bit lost (see #1317)
RUN chmod 755 /otelcol

FROM alpine:${ALPINE_VERSION} AS smartagent
ARG SMART_AGENT_RELEASE_URL
RUN  wget -O /signalfx-agent.tar.gz "$SMART_AGENT_RELEASE_URL" && \
     tar -xzf /signalfx-agent.tar.gz && \
     /signalfx-agent/bin/patch-interpreter /signalfx-agent && \
     rm -f /signalfx-agent/bin/signalfx-agent /signalfx-agent/bin/agent-status /signalfx-agent.tar.gz

FROM scratch
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=otelcol /otelcol /

ENV SPLUNK_BUNDLE_DIR="/usr/lib/splunk-otel-collector/agent-bundle"
COPY --from=smartagent /signalfx-agent $SPLUNK_BUNDLE_DIR
ENV SPLUNK_COLLECTD_DIR=$SPLUNK_BUNDLE_DIR/run/collectd

COPY config/collector/gateway_config.yaml    /etc/otel/collector/gateway_config.yaml
COPY config/collector/otlp_config_linux.yaml /etc/otel/collector/otlp_config_linux.yaml
COPY config/collector/agent_config.yaml      /etc/otel/collector/agent_config.yaml

ENV SPLUNK_CONFIG=/etc/otel/collector/agent_config.yaml

ENTRYPOINT ["/otelcol"]
EXPOSE 13133 14250 14268 55680 4317 6060 7276 8888 9411 9443
