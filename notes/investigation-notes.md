# Investigation Notes — Wazuh Notification Delivery Validation

## 1. Investigation Objective

The objective was to determine whether a controlled FIM event on a Windows endpoint could be traced through Wazuh alert processing and whether the evidence was sufficient to confirm notification delivery.

The investigation deliberately separates confirmed observations from unverified notification outcomes.

## 2. Endpoint and FIM Configuration

The Windows endpoint was identified as `DESKTOP-9MMM37V`, with Wazuh agent ID `001`.

The local agent configuration was inspected at:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

The configuration contained the following directory entry:

```xml
<directories realtime="yes">C:\FIM-Lab</directories>
```

This established that `C:\FIM-Lab` was configured for realtime FIM monitoring.

The initial test-file creation attempt failed because the local directory did not exist. The error was:

```text
Could not find a part of the path
'C:\FIM-Lab\lab118-notification-test.txt'.
```

The directory was then made available, allowing the controlled test file to be created and modified.

## 3. Controlled Test

The test file was:

`C:\FIM-Lab\lab118-notification-test.txt`

The first successful file creation produced the following recorded properties:

- File size: 83 bytes
- Last modification time: 26 October 2026, 06:58:24

The file was subsequently modified by appending a second test message.

A SHA-256 hash was calculated after the modification:

`5D00B51596864685E021670EA6617ECE96FBAF1D21060A1F02ACB3314345A99E`

This hash records the file state at the time the command was run. It should not be treated as the hash reported by the Wazuh event unless the values are explicitly compared.

## 4. Observed Wazuh Event

A FIM event was visible in the Wazuh Dashboard with these details:

| Field | Observed value |
|---|---|
| Timestamp | `2026-10-26T06:58:36.657` as displayed |
| Index | `wazuh-alerts-4.x-2026.10.26` |
| Agent ID | `001` |
| Agent name | `DESKTOP-9MMM37V` |
| Agent IP | `192.168.203.1` |
| Decoder | `syscheck_integrity_changed` |
| Location | `syscheck` |
| Rule description | `Integrity checksum changed.` |
| FIM mode | `realtime` |
| Changed attributes | `size,mtime,md5,sha1,sha256` |
| Size change | `83` to `151` |
| Rule fired count | `1` |

The full log identified the file as:

`c:\fim-lab\lab118-notification-test.txt`

The event reported that the file was modified and that its integrity-related attributes changed.

This is evidence that Wazuh recorded a FIM integrity-change event for the test file.

## 5. Search and Correlation Findings

A broad query against `wazuh-alerts-*` returned recent alerts associated with the Windows endpoint.

A targeted query then filtered on:

- Agent ID `001`
- Exact `syscheck.path` value
- A recent timestamp range

The targeted query returned zero hits.

A subsequent query using the same exact-path filter also returned zero hits, even though the dashboard displayed a FIM event whose full log contained the test file path.

This is an unresolved discrepancy. Possible investigation areas include the actual indexed field structure, path normalization, the selected time range, and whether the visible event's path is stored in `syscheck.path` in the expected format.

The available evidence does not establish which explanation is responsible.

## 6. Manager and Indexer Observations

Manager logs contained warnings at approximately 06:35 on 26 October 2026, including:

- Indexer connector initialization failures for several `wazuh-states-inventory-*` indices.
- Repeated messages stating that agent `001` failed to sync with the indexer.

Subsequent messages around 06:42 reported successful initialization for several inventory indices, including protocols, networks, processes, hardware, and system information.

These observations show that indexer-related issues occurred during the broader investigation period. They do not prove that the warnings caused the FIM query mismatch or affected notification delivery.

## 7. Notification Delivery Assessment

The following stages must be distinguished:

| Stage | Assessment |
|---|---|
| Controlled file modification | Confirmed |
| FIM integrity-change event visible in Wazuh | Confirmed |
| Exact-path query successfully returning the event | Not confirmed |
| Notification mechanism and configuration | Not verified |
| Notification eligibility for the test alert | Not verified |
| Notification send attempt | Not verified |
| Destination receipt | Not verified |

The investigation did not establish whether email or another notification integration was configured for this deployment.

No destination-side evidence was collected to demonstrate receipt of a notification.

## 8. Evidence-Based Conclusion

The available evidence confirms that the controlled file modification produced a Wazuh FIM integrity-change event.

It does not confirm that the event triggered a notification, that a notification was sent, or that the intended destination received it.

The correct overall classification is **Inconclusive**.

Further work should focus on resolving the indexed-field discrepancy and inspecting the actual notification configuration and delivery records.
