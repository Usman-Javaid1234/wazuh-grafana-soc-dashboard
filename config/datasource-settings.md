# Grafana Data Source Settings

Data source type: OpenSearch (plugin: `grafana-opensearch-datasource`)

Install the plugin if it is missing:

```
grafana-cli plugins install grafana-opensearch-datasource
```

Restart Grafana after installing it.

## HTTP

| Setting | Value |
|---|---|
| URL | `https://<WAZUH_VM_IP>:9200` |
| Access | Server (default) |
| Timeout | Not set |

Port 9200 is the default HTTPS port of the Wazuh Indexer (OpenSearch) REST API.
Grafana sends all of its queries to this port.

## Auth

| Setting | Value |
|---|---|
| Basic auth | On |
| User | Your Wazuh Indexer user (for example `admin`) |
| Password | Your Wazuh Indexer password |
| Skip TLS Verify | On (the Indexer uses a self-signed certificate) |
| TLS Client Auth | Off |
| With CA Cert | Off |
| With Credentials | Off |
| Forward OAuth Identity | Off |

Use your own Indexer credentials. Do not commit them to the repository.

## OpenSearch details

| Setting | Value |
|---|---|
| Index name | `wazuh-alerts-*` |
| Time field name | `@timestamp` |
| Version | OpenSearch (compatibility mode) |
| Serverless | Off |
| Max concurrent shard requests | 5 |
| Min time interval | 10s |
| PPL enabled | On |

Click **Get Version and Save** after entering the details, then **Save & test**.

## Logs

| Setting | Value |
|---|---|
| Message field name | `_source` |
| Level field name | Not set |

## Panel queries

All four panels use Query type **Lucene**, Index `wazuh-alerts-*`, and metric **Count**.

| Panel | Group by | Field | Size | Visualization |
|---|---|---|---|---|
| Security Alerts Over Time | Date Histogram | `@timestamp` | Auto interval | Time series |
| Alerts by Severity Level | Terms | `rule.level` | 20 | Bar chart |
| Top 10 Alert Types | Terms | `rule.description` | 10 | Bar chart (horizontal) |
| Alerts by Agent | Terms | `agent.name` | 20 | Pie chart |

The Alerts by Severity Level panel also uses a **Convert field type** transformation that
converts `rule.level` to a string. The bar chart needs a string or time field on its category
axis, and `rule.level` is numeric.
