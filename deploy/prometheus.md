# Prometheus metrics

Prometheus is quite popular now, so there is a 3rd party exporter implementation for Centrifugo
[centrifugo_exporter](https://github.com/nordicdyno/centrifugo_exporter)

Centrifugo [has it's own metrics exporting mechanism in JSON format](../server/stats.html). It is possible for this exporter logic to migrate to Centrifugo code to avoid duplication.

For now this exporter can be a temporary or permanent solution for people with Prometheus or other monitoring software with [OpenMetrics](https://github.com/RichiH/OpenMetrics) exporting format support.
