# Detection Test Results

## GET/HEAD comparison

The client at `10.77.10.10` sent two HTTP requests to `10.77.20.20:8080`. The monitor captured the traffic on `client0`, and Suricata 7.0.3 analyzed the saved capture with the custom rule in [local.rules](../configs/suricata/local.rules).

| Request | HTTP status | Flow ID | Alerts for SID 1000001 |
|---|---|---|---|
| GET `/` | 200 | 1137104546393794 | 1 |
| HEAD `/` | 200 | 876994957130303 | 0 |

The GET alert and GET HTTP record share a flow ID. The HEAD transaction appears in the HTTP log without an alert for this SID. This is a positive/negative test of these two methods; it does not establish behavior for every possible request or method.

### Files

- [Original GET/HEAD packet capture](../evidence/pcaps/get-head.pcap)
- [Selected HTTP and alert records](../evidence/logs/get-head-events.jsonl)
- [Custom detection rule](../configs/suricata/local.rules)

The JSONL file retains the original event fields for HTTP events and alerts with SID 1000001. Other event types were excluded. It is an extract, not the complete EVE log.

### Test commands

On the client, with the lab HTTP server running:

```bash
curl --noproxy '*' --max-time 10 -i http://10.77.20.20:8080/
curl --noproxy '*' --max-time 10 -I http://10.77.20.20:8080/
```

To inspect the exported results from the repository root:

```bash
jq 'select(.event_type == "http") | {flow_id, method: .http.http_method, status: .http.status}' evidence/logs/get-head-events.jsonl
jq 'select(.event_type == "alert") | {flow_id, sid: .alert.signature_id, method: .http.http_method}' evidence/logs/get-head-events.jsonl
```

Suricata's `alert` action records a detection; it does not block the request. The alert record reports `action: allowed`.

## Service state

The exported service records show Suricata active and enabled on the monitor at collection time. The status output identifies version 7.0.3 and the AF_PACKET service command.

- [Active state](../evidence/logs/suricata-active.txt)
- [Enabled state](../evidence/logs/suricata-enabled.txt)
- [Service status](../evidence/logs/suricata-status.txt)

Live alerting and alerting after reboot were also verified during the lab. The service-state export alone does not demonstrate a newly generated alert after that boot; the GET/HEAD records above are from the earlier offline capture.

## Evidence dates

- GET/HEAD events: September 28, 2026, approximately 19:09 UTC.
- Configuration/service export: October 14, 2026, approximately 06:02 UTC (October 13, approximately 23:02 Pacific Daylight Time).

The capture and configuration export were collected at different times. Firewall counters in the later export are a snapshot, not counters from the original HTTP test.
