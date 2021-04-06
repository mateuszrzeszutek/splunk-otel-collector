ARG SMART_AGENT_VERSION_NUMBER

FROM alpine:3.12 as certs
RUN apk --update add ca-certificates

FROM alpine:3.12 AS otelcol
COPY otelcol /
# Note that this shouldn't be necessary, but in some cases the file seems to be
# copied with the execute bit lost (see #1317)
RUN chmod 755 /otelcol

FROM quay.io/signalfx/signalfx-agent:${SMART_AGENT_VERSION_NUMBER}
ENV SPLUNK_CONFIG=/etc/otel/collector/agent_config.yaml
ENV SPLUNK_BUNDLE_DIR=/usr/lib/splunk-otel-collector/agent-bundle
ENV SPLUNK_COLLECTD_DIR=/usr/lib/splunk-otel-collector/agent-bundle/run/collectd
RUN mkdir -p /usr/lib/splunk-otel-collector && \
    ln -sf /usr/lib/signalfx-agent /usr/lib/splunk-otel-collector/agent-bundle
COPY --from=certs   /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=otelcol /otelcol /
COPY config/collector/gateway_config.yaml    /etc/otel/collector/gateway_config.yaml
COPY config/collector/otlp_config_linux.yaml /etc/otel/collector/otlp_config_linux.yaml
COPY config/collector/agent_config.yaml      /etc/otel/collector/agent_config.yaml
ENTRYPOINT ["/otelcol"]
EXPOSE 13133 14250 14268 55680 4317 6060 7276 8888 9411 9443
