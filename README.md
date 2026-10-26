# Lab 118 — Wazuh Notification Delivery Validation

## Overview

This lab examines the notification-delivery validation process in a Wazuh-based SOC environment. The objective is to distinguish between an endpoint event being generated, a corresponding alert being indexed, a notification being attempted, and the intended destination confirming receipt.

A Wazuh alert appearing in the index does not independently prove that an email or integration notification was sent or received.

## Objectives

- Generate a controlled file modification in a directory monitored by Wazuh File Integrity Monitoring (FIM).
- Verify whether Wazuh detects the change and records an integrity alert.
- Investigate discrepancies between an observed alert and exact-path searches.
- Review available evidence of manager and indexer health.
- Identify what is required to verify notification configuration, delivery attempts, and destination receipt.
- Document findings and limitations without assuming that alert generation proves successful notification delivery.

## Environment

- **SIEM / Endpoint Security:** Wazuh
- **Endpoint OS:** Windows
- **Endpoint hostname:** `DESKTOP-9MMM37V`
- **Wazuh agent ID:** `001`
- **Agent IP shown in the event:** `192.168.203.1`
- **Alert index:** `wazuh-alerts-4.x-2026.10.26`
- **Monitored directory:** `C:\FIM-Lab`
- **Test file:** `C:\FIM-Lab\lab118-notification-test.txt`
- **FIM monitoring mode:** Realtime

## Investigation Scenario

A controlled file modification was used to test whether Wazuh could detect a change in its configured FIM directory. The resulting event was examined in the Wazuh Dashboard.

The investigation then compared the observed event with targeted searches against `wazuh-alerts-*`. Those searches returned zero matches for the exact `syscheck.path` filter, despite an event being visible in the dashboard with the test file path in its full log.

Manager logs also contained indexer initialization and agent synchronization warnings, followed by successful initialization messages for several inventory indices.

The available evidence confirms that a FIM integrity-change event was recorded. However, the notification configuration, notification attempt, and destination receipt were not verified.

## Procedure

1. Inspect the Windows agent configuration and identify existing FIM directories.
2. Confirm that `C:\FIM-Lab` exists.
3. Create or modify the controlled test file.
4. Inspect recent Wazuh alerts and identify the matching FIM event.
5. Search for the event using agent ID, timestamp, and file path.
6. Investigate query mismatches and manager/indexer warnings.
7. Review the configured notification mechanism and its eligibility conditions, if administrative access is available.
8. Check delivery records and verify receipt at the intended destination.
9. Record the final result based only on available evidence.

## Key Findings

- The agent configuration contained a `C:\FIM-Lab` directory entry with realtime monitoring enabled.
- The first attempt to create the test file failed because the directory did not exist locally.
- After the directory was made available, the test file was created and subsequently modified.
- A Wazuh event showed the test file being modified in realtime mode.
- The event reported changes to size, modification time, MD5, SHA-1, and SHA-256 attributes.
- The event description was `Integrity checksum changed.`
- Exact-path searches against `syscheck.path` returned zero hits, despite the event visible in the dashboard.
- Manager logs showed indexer initialization warnings and failed agent synchronization attempts, followed by successful initialization messages for several inventory indices.
- No evidence was collected that confirms successful notification delivery to an intended destination.

## Result

**Overall outcome: Inconclusive**

The FIM detection stage was demonstrated by the recorded integrity-change event. The end-to-end notification-delivery objective was not completed because notification configuration, send status, and destination receipt were not established.

This result does not mean notification delivery failed. It means the available evidence is insufficient to confirm success or failure.

## Limitations

- Notification settings were not confirmed.
- The intended notification destination was not established.
- Notification send attempts and destination-side receipt were not verified.
- The discrepancy between the visible event and exact-path search results remains unresolved.
- Indexer warnings were observed, but their direct relationship to the query mismatch or notification delivery was not established.

## Conclusion

Reliable notification validation requires evidence across the complete workflow: event generation, alert recording, notification eligibility, send attempt, and destination receipt.

This investigation confirmed a controlled FIM integrity-change event but did not establish end-to-end notification delivery. The remaining work is to verify the configured notification mechanism and obtain delivery evidence.
