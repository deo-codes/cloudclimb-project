# CloudClimb Project 01 — Week 5 KQL Queries

These six queries were retrieved from Azure Log Analytics query history. Relative `ago(...)` windows refer to the time each query is run; older events may not appear when rerun.

## Query 1 — Authentication Syslog Events

Displays recent authentication-related Syslog entries.

```kql
Syslog
| where TimeGenerated > ago(30m)
| where Facility in ("auth", "authpriv")
| project TimeGenerated, Computer, Facility,
          SeverityLevel, ProcessName, SyslogMessage
| order by TimeGenerated desc
| take 20
```

## Query 2 — Syslog Event Count by Computer

Counts Syslog records by computer over the last 30 minutes.

```kql
Syslog
| where TimeGenerated > ago(30m)
| summarize EventCount = count() by Computer
| order by EventCount desc
```

## Query 3 — Synthetic SSH Monitoring Test

Checks whether the CloudClimb synthetic SSH test marker was ingested. The earlier test returned two matching rows; this is not evidence of a brute-force attack.

```kql
Syslog
| where TimeGenerated > ago(2h)
| where Computer == "vm-app-dev-eastus"
| where SyslogMessage contains "CloudClimb-Week5-SSH-Monitoring-Test"
| project TimeGenerated, Computer, Facility,
          SeverityLevel, ProcessName, SyslogMessage
| order by TimeGenerated desc
```

## Query 4 — Authentication Events by Facility and Process

Summarizes authentication-related events by facility and process for the application VM.

```kql
Syslog
| where TimeGenerated > ago(24h)
| where Computer == "vm-app-dev-eastus"
| where Facility in~ ("auth", "authpriv")
| summarize EventCount = count()
    by Facility, ProcessName
| order by EventCount desc
```

## Query 5 — SSH Service Activity

Lists SSH-related Syslog messages for the application VM.

```kql
Syslog
| where TimeGenerated > ago(24h)
| where Computer == "vm-app-dev-eastus"
| where ProcessName in~ ("sshd", "ssh")
    or SyslogMessage contains "sshd"
| project TimeGenerated, Computer,
          Facility, SeverityLevel,
          ProcessName, SyslogMessage
| order by TimeGenerated desc
| take 30
```

## Query 6 — Failed Authentication Attempts

Looks for common failed-login messages; a matching result is not guaranteed.

```kql
Syslog
| where TimeGenerated > ago(24h)
| where SyslogMessage contains "Failed password"
    or SyslogMessage contains "authentication failure"
| project TimeGenerated, Computer,
          ProcessName, SyslogMessage
| order by TimeGenerated desc
```
