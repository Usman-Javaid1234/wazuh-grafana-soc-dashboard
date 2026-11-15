# Validation Commands

These commands compare what the Grafana dashboard shows with the raw alert log on the Wazuh server. Run them over SSH on the Wazuh VM.

## Before you compare

- `/var/ossec/logs/alerts/alerts.json` holds the current day's alerts. The Grafana panels cover whatever time range is selected (this dashboard uses the last 6 hours). The numbers only line up when that range covers all the alerts in the file.
- The Indexer can lag a few moments behind the log. If the numbers differ slightly, wait a minute, refresh the panel, and compare again.
- Older days are rotated into compressed files under `/var/ossec/logs/alerts/<year>/<month>/`. Use `zgrep` instead of `grep` for those.

## 1. Alerts at severity level 3

Count the lines in the log that contain a level 3 rule:

```bash
sudo grep -c '"rule":{"level":3' /var/ossec/logs/alerts/alerts.json
```

Result recorded for this project: `479`

Compare it with the bar for level 3 in the **Alerts by Severity Level** panel. Grafana showed 479 once indexing had caught up (it showed 471 on the first check).

## 2. Total number of alerts

```bash
sudo wc -l /var/ossec/logs/alerts/alerts.json
```

Result recorded for this project: `1030`

## 3. Alerts per agent

Count how many alerts name each agent:

```bash
sudo cat /var/ossec/logs/alerts/alerts.json | grep -o '"agent":{"[^}]*name":"[^"]*"' | grep -o '"name":"[^"]*"' | sort | uniq -c
```

Result recorded for this project:

```
    920 "name":"SUIII"
    113 "name":"wazuh-server"
```

That is about 89% and 11%. The **Alerts by Agent** pie chart showed 90% and 10%, which matches after rounding.

The agent total (1033) is slightly higher than the line count (1030). The pattern counts matches, not alerts, so a few alerts can match more than once. The `jq` version below counts one agent name per alert.

## 4. Stricter check with jq (optional)

Install `jq` with your package manager (`sudo dnf install -y jq` or `sudo yum install -y jq`), then:

```bash
sudo jq -r '.agent.name' /var/ossec/logs/alerts/alerts.json | sort | uniq -c
sudo jq -r 'select(.rule.level == 3) | .rule.level' /var/ossec/logs/alerts/alerts.json | wc -l
```

## 5. Ask the Indexer directly (optional)

This compares Grafana with the Indexer itself, using the same data Grafana queries. These commands were not part of the recorded results. If one fails, check the quoting and the credentials.

On the Wazuh VM:

```bash
curl -k -u <INDEXER_USER>:<INDEXER_PASSWORD> \
  "https://localhost:9200/wazuh-alerts-*/_count?q=rule.level:3"
```

Limit the count to the same window as the dashboard (last 6 hours):

```bash
curl -k -u <INDEXER_USER>:<INDEXER_PASSWORD> -G \
  "https://localhost:9200/wazuh-alerts-*/_count" \
  --data-urlencode "q=rule.level:3 AND @timestamp:[now-6h TO now]"
```

From Windows PowerShell, use `curl.exe` and the VM address:

```powershell
curl.exe -k -u <INDEXER_USER>:<INDEXER_PASSWORD> "https://<WAZUH_VM_IP>:9200/wazuh-alerts-*/_count?q=rule.level:3"
```

## Summary of results

| Check | Grafana | Raw log | Result |
|---|---|---|---|
| `rule.level` 3 count | 479 | 479 | Match |
| Agent split | 90% / 10% | 920 / 113 (about 89% / 11%) | Match after rounding |
| Total alerts in `alerts.json` | Not shown on the dashboard | 1030 | Reference only |
