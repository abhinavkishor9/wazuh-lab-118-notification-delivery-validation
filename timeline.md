# Timeline — Wazuh Notification Delivery Validation

**Investigation date:** 26 October 2026

This timeline records the observed sequence. Where an exact timestamp was not captured, the entry is marked as unrecorded rather than estimated.

| Time / Sequence | Activity | Observation | Status |
|---|---|---|---|
| Before testing | Reviewed the Windows agent configuration | `C:\FIM-Lab` was listed as a realtime FIM directory. | Confirmed |
| Initial test attempt | Attempted to create the test file | PowerShell reported that `C:\FIM-Lab\lab118-notification-test.txt` could not be created because the path did not exist. | Failed |
| After initial failure | Checked and made the test directory available | The directory was created or otherwise made available before the successful file creation. | Resolved |
| 26 Oct 2026, 06:58:24 | Created the test file | `C:\FIM-Lab\lab118-notification-test.txt` was recorded as 83 bytes. | Confirmed |
| After file creation | Modified the test file | A second controlled message was appended, and a SHA-256 hash was calculated. | Confirmed |
| 26 Oct 2026, 06:58:36.657 | Examined the FIM event in the dashboard | Wazuh showed a realtime integrity-change event for the test file. The size changed from 83 to 151, and size, modification time, MD5, SHA-1, and SHA-256 attributes were reported as changed. | Confirmed |
| During event searches | Queried `wazuh-alerts-*` by agent and time | Broad queries returned recent events for the Windows endpoint. | Confirmed |
| During event searches | Queried by agent ID and exact `syscheck.path` | The query returned zero hits, despite the visible FIM event. | Unresolved discrepancy |
| 26 Oct 2026, approximately 06:35–06:36 | Reviewed manager logs | Indexer initialization warnings and failed synchronization attempts for agent `001` were observed. | Confirmed |
| 26 Oct 2026, approximately 06:42 | Reviewed later manager logs | Several inventory index initialization messages reported success. | Confirmed |
| End of investigation | Assessed notification evidence | Notification configuration, send attempt, and destination receipt were not verified. | Inconclusive |

## Timeline Interpretation

The FIM event was visible shortly after the recorded test-file creation and modification. This supports the conclusion that Wazuh detected a file integrity change.

The exact-path search discrepancy was not resolved during the investigation. Indexer-related warnings were observed, followed by successful initialization messages for several inventory indices, but no direct causal relationship was established.

The investigation did not verify whether the alert triggered a notification or whether the intended destination received one.

## Final Status

**Overall result: Inconclusive**

FIM event generation was demonstrated. End-to-end notification delivery remains unverified.
