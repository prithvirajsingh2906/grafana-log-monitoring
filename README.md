# grafana-log-monitoring
Centralized application log monitoring using WinSCP, Grafana Alloy, Grafana Loki, and Grafana dashboards.

# WinSCP to Grafana — Centralized Log Monitoring

A centralized application log monitoring solution using **WinSCP, Grafana Alloy, Grafana Loki, and Grafana** to collect, process, query, and visualize application logs from remote servers through a Windows-based monitoring environment.

## Overview

Troubleshooting application issues often requires manually connecting to remote servers, downloading log files, and searching through large amounts of text. This project simplifies that workflow by creating a centralized log monitoring pipeline and providing dashboards for log analysis.

The solution synchronizes remote application logs to a local Windows environment, forwards them to Grafana Loki through Grafana Alloy, and visualizes the collected logs through Grafana dashboards.

## Architecture

```text
Remote Linux Servers
        |
        | SSH / SFTP
        v
      WinSCP
        |
        | Log synchronization
        v
Windows Log Staging Directory
        |
        v
    Grafana Alloy
        |
        | Push log entries
        v
    Grafana Loki
        |
        | LogQL queries
        v
      Grafana
        |
        v
Application Monitoring Dashboard
```

**Note:** The current architecture uses local Windows storage as a staging area. Direct collection from remote servers or another centralized ingestion approach can be evaluated to reduce local disk usage.

## Technology Stack

- **WinSCP:** Secure file transfer and remote log synchronization.
- **Grafana Alloy:** Log collection and forwarding.
- **Grafana Loki:** Centralized log storage and querying.
- **Grafana:** Dashboard visualization and log exploration.
- **Windows Batch Scripting:** Automating synchronization and monitoring startup.
- **Linux / SSH / SFTP:** Accessing remote application log files.
- **LogQL:** Querying and analyzing log data.

## Key Features

- Centralized visibility into application logs from multiple servers.
- Application and server-based filtering using log labels.
- Real-time-style log exploration and historical log analysis.
- Log event counts and activity comparisons across servers.
- Identification of frequently active log files.
- Search for critical errors, fatal events, memory issues, and connection failures.
- Configurable log synchronization and collection scripts.
- Grafana dashboards for operational monitoring and troubleshooting.

## Dashboard Capabilities

The Application Monitoring dashboard can provide:

- **Application Filter:** Focus on a selected application.
- **Server Filter:** Narrow log analysis to a particular server.
- **Log Events:** Count log entries during the selected time range.
- **Log Volume:** Estimate the amount of log data processed.
- **Error Analysis:** Identify and investigate error patterns.
- **Live Logs:** Explore recent log entries.
- **Log Activity by Server:** Compare log activity across servers.
- **Top Log Files:** Identify files generating the most log entries.

Dashboard panels depend on the configured labels, available log fields, and the queries used.

## Example LogQL Queries

### 1. Count log events

```logql
sum(
  count_over_time(
    {app=~"$application", server=~"$server"}[$__range]
  )
)
```

Counts log entries matching the selected application and server filters during the dashboard's selected time range.

### 2. View recent logs

```logql
{app=~"$application", server=~"$server"}
```

Displays log entries matching the selected filters.

### 3. Log activity by server

```logql
sum by (server) (
  count_over_time(
    {app=~"$application", server=~"$server"}[$__range]
  )
)
```

Compares log event counts by server.

### 4. Top log files by activity

```logql
topk(
  10,
  sum by (filename) (
    count_over_time(
      {app=~"$application", server=~"$server"}[$__range]
    )
  )
)
```

Identifies the ten most active log files based on log entry counts.

### 5. Search for critical events

```logql
sum(
  count_over_time(
    {app=~"$application", server=~"$server"}
      |~ "(?i)critical|fatal|panic|out.of.memory|connection refused"
      [$__range]
  )
)
```

Counts matching log entries containing selected critical keywords. This is a keyword-based search, so its accuracy depends on the application's log format and terminology.

## Prerequisites

- Windows machine or Windows virtual machine.
- Access to remote Linux servers through an authorized SSH/SFTP account.
- WinSCP.
- Grafana Alloy.
- Grafana Loki.
- Grafana.
- Required network access and permissions.
- Sufficient disk space for temporary or synchronized logs.

Use compatible versions of the components and validate their configuration before deployment.

## Installation and Setup

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/winscp-grafana-log-monitoring.git
cd winscp-grafana-log-monitoring
```

### 2. Configure WinSCP

Configure an authorized remote session and the log synchronization paths. Use environment-specific settings and do not commit passwords, private keys, or session exports containing credentials.

### 3. Configure Grafana Alloy

Configure Alloy to read the synchronized log files, attach useful labels such as `app` and `server`, and forward log entries to Loki.

### 4. Configure Grafana Loki

Set the appropriate storage directory, retention settings, and ingestion configuration for the deployment environment.

### 5. Start Grafana

Start Loki and Alloy first, then Grafana. Confirm that the Loki data source is healthy.

### 6. Import the dashboard

Open Grafana, navigate to Dashboards, import the dashboard JSON from the `dashboards/` directory, and select the configured Loki data source.

### 7. Validate the pipeline

Verify that:
- WinSCP can synchronize the intended log files.
- Alloy discovers and reads the files.
- Loki accepts and stores log entries.
- Grafana queries return results for the selected time range.
- Application and server filters work correctly.

See `docs/installation.md` for the detailed environment-specific setup.

## Troubleshooting

| Problem | What to check |
|---|---|
| No logs in Grafana | Verify synchronization, Alloy targets, Loki readiness, and the time range. |
| Loki data source error | Check the Loki URL, service status, port, and connectivity. |
| Slow dashboard | Reduce the query time range, avoid unnecessary regex filters, and optimize dashboard refresh intervals. |
| Missing critical events | Check log wording, label filters, timestamps, and query syntax. |
| High local disk usage | Review synchronization scope, retention, and log staging cleanup policies. |
| Missing application labels | Validate Alloy processing rules and the labels attached to each stream. |

## Security Considerations

- Never commit real credentials, private keys, tokens, or production logs.
- Replace internal server names and IP addresses with placeholders in documentation.
- Use `.gitignore` to exclude local logs, generated data, and sensitive configuration files.
- Restrict Loki and Grafana network access appropriately.
- Configure authentication and retention according to the deployment environment.
- Review exported dashboards and configuration files for sensitive information before publishing.

## Future Enhancements

- Reduce dependency on local log staging storage.
- Improve critical-event detection and alerting.
- Add configurable alerts for repeated failures and abnormal log activity.
- Improve query performance for large time ranges.
- Automate service health checks and log synchronization validation.
- Deploy the monitoring stack on a dedicated Windows virtual machine.
- Add CI/CD validation for sanitized configuration files and dashboard JSON.

## Learning Outcomes

This project demonstrates practical experience with centralized logging, observability, Linux-to-Windows log transfer, LogQL queries, dashboard design, Windows scripting, infrastructure troubleshooting, and monitoring pipeline configuration.

## License

This project is intended as a technical portfolio and learning resource. Add a license only after confirming that you have permission to publish the included code and documentation.
