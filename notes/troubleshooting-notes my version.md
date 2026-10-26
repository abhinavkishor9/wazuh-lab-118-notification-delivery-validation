# Troubleshooting Notes 

## 1. Test File Creation Failed

### Symptom

The initial PowerShell command failed with:

```text
Could not find a part of the path
'C:\FIM-Lab\lab118-notification-test.txt'.
```

The subsequent `Get-Item` command also failed because the file did not exist.

### Investigation

The Wazuh agent configuration contained a `C:\FIM-Lab` directory entry with realtime monitoring enabled. However, a configured monitoring path does not guarantee that the directory currently exists on the endpoint.

### Resolution

Check whether the directory exists:

```powershell
Test-Path "C:\FIM-Lab"
```

If the result is `False`, create the directory:

```powershell
New-Item -Path "C:\FIM-Lab" -ItemType Directory
```

Then create the test file:

```powershell
$testFile = "C:\FIM-Lab\lab118-notification-test.txt"

"Lab 118 notification delivery validation test - $(Get-Date -Format o)" |
    Set-Content -Path $testFile

Get-Item $testFile |
    Select-Object FullName, Length, LastWriteTime
```

### Lesson

Configuration inspection and filesystem validation are separate checks. Confirm that the configured directory exists before attempting to create a test file.

## 2. Exact-Path Search Returned Zero Hits

### Symptom

A query filtering on agent ID `001` and the exact `syscheck.path` value returned zero hits, although the dashboard displayed a FIM event containing the test file path.

### Investigation

The visible event's full log used this path:

`c:\fim-lab\lab118-notification-test.txt`

The query used the following field filter:

```json
{
  "term": {
    "syscheck.path": "C:\\FIM-Lab\\lab118-notification-test.txt"
  }
}
```

The difference in displayed path casing may be worth checking, but it does not by itself establish the cause. The event's full log and the structured `syscheck.path` field are not necessarily identical representations.

### Troubleshooting Query 1: Search by Agent and Time

Use Wazuh Dashboard → Dev Tools:

```json
GET wazuh-alerts-*/_search
{
  "size": 50,
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "agent.id": "001"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h",
              "lte": "now"
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

Inspect the returned events for the test file path and the `syscheck` fields.

### Troubleshooting Query 2: Inspect Candidate Events

If the broad search returns too many results, filter on the agent and FIM-related event description:

```json
GET wazuh-alerts-*/_search
{
  "size": 50,
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "agent.id": "001"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h",
              "lte": "now"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "full_log": "lab118-notification-test.txt"
          }
        },
        {
          "match_phrase": {
            "rule.description": "Integrity checksum changed."
          }
        }
      ],
      "minimum_should_match": 1
    }
  },
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

This is a candidate-event search, not proof that the exact structured path filter is correct. If the `full_log` field is not searchable in the deployed mapping, inspect the returned event fields or use the Dashboard's event details.

### Troubleshooting Query 3: Inspect the Returned Field Structure

Once the candidate event is located, inspect its complete source document. Check whether the path appears in `syscheck.path`, `full_log`, or another field.

Do not assume the correct field name or alter the query solely to force a match.

### Lesson

A zero-hit query means the document did not match that query as executed. It does not prove that the event was never generated or indexed.

## 3. Indexer Connector Warnings

### Symptom

Manager logs showed warnings about failed initialization of several `wazuh-states-inventory-*` indices and failed synchronization of agent `001`.

### Later Observation

Subsequent log entries reported successful initialization for several inventory indices.

### Troubleshooting Commands

If authorized access to the Wazuh manager is available, inspect the relevant log entries:

```bash
sudo grep -iE 'indexer-connector|failed to sync agent|initialization failed' /var/ossec/logs/ossec.log | tail -n 100
```

Inspect the most recent manager log entries:

```bash
sudo tail -n 200 /var/ossec/logs/ossec.log
```

Check whether relevant warnings continued after the successful initialization messages.

### Lesson

The warnings establish that indexer-related problems occurred. They do not independently establish the cause of the query mismatch or demonstrate a notification-delivery failure.

## 4. Notification Delivery Could Not Be Verified

### Symptom

A FIM integrity-change event was visible, but there was no verified evidence of notification receipt.

### Required Checks

1. Establish which notification mechanism is configured.
2. Inspect its destination and eligibility filters.
3. Determine whether the test alert qualified for notification.
4. Find a corresponding send attempt or delivery record.
5. Verify receipt at the intended destination.

### Important Distinction

- An indexed alert proves that an alert was recorded.
- A notification configuration entry does not prove that a notification was sent.
- A send attempt does not necessarily prove successful delivery.
- Destination-side receipt provides evidence that the notification arrived.

If the notification mechanism or delivery records are inaccessible, document that limitation instead of assuming success or failure.

