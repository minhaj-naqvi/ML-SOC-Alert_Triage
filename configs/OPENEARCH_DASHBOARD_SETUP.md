# OpenSearch / Wazuh Dashboard Setup

No additional OpenSearch configuration file was required.

ML-enriched alerts are written to:

`/var/ossec/logs/alerts/alerts_ml.jsonl`

The provided `filebeat.yml` forwards these events to the Wazuh Indexer/OpenSearch instance using the daily index pattern:

`wazuh-ml-alerts-*`

Within the Wazuh Dashboard, create a data view/index pattern for:

`wazuh-ml-alerts-*`

This data view can then be used to create visualizations and a dedicated dashboard for the ML-enriched alerts.

See `filebeat.yml` for the corresponding ingestion configuration.
