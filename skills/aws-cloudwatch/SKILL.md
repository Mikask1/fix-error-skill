---
name: aws-cloudwatch
description: Retrieve and query AWS CloudWatch logs using the AWS CLI. Use when you need to fetch log groups, log streams, log events, or run CloudWatch Insights queries to find errors, filter by pattern, or analyze log volume.
---

# AWS CloudWatch Logs

Retrieve logs from AWS CloudWatch using the AWS CLI.

## Pre-flight: verify credentials

```bash
aws sts get-caller-identity
```

If this fails, the CLI is not configured. Tell the user to run `aws configure` or `aws configure sso` and retry. Do not proceed until credentials are confirmed.

Add `--region <region>` to any command below when logs are in a non-default region.

---

## Operations

### List log groups

```bash
# All log groups
aws logs describe-log-groups --query 'logGroups[*].logGroupName' --output table

# Filtered by prefix
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/my-service" \
  --query 'logGroups[*].logGroupName' --output table
```

### List recent log streams

```bash
aws logs describe-log-streams \
  --log-group-name "<LOG_GROUP>" \
  --order-by LastEventTime \
  --descending \
  --max-items 10 \
  --query 'logStreams[*].{name:logStreamName,last:lastEventTimestamp}'
```

### Fetch log events from a stream

```bash
aws logs get-log-events \
  --log-group-name "<LOG_GROUP>" \
  --log-stream-name "<LOG_STREAM>" \
  --limit 200 \
  --query 'events[*].{time:timestamp,msg:message}'
```

Add `--start-time <epoch-ms>` and `--end-time <epoch-ms>` to restrict the window.

### Filter log events by pattern (across a group)

```bash
aws logs filter-log-events \
  --log-group-name "<LOG_GROUP>" \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s000) \
  --end-time $(date +%s000) \
  --limit 100
```

Pattern syntax: `"ERROR"` (substring), `?"ERROR" ?"WARN"` (OR), `{ $.level = "ERROR" }` (JSON field).

### CloudWatch Insights query (best for large volumes)

**Step 1 — Start the query:**
```bash
aws logs start-query \
  --log-group-name "<LOG_GROUP>" \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message
    | filter @message like /ERROR|Exception|FATAL/
    | sort @timestamp desc
    | limit 50'
```

Save the returned `queryId`.

**Step 2 — Poll for results:**
```bash
aws logs get-query-results --query-id "<QUERY_ID>"
```

Re-run until `status` is `Complete`. Results are in the `results` array.

**Useful Insights query patterns:**

```
# Top error messages by frequency
fields @message
| filter @message like /ERROR/
| stats count() as hits by @message
| sort hits desc
| limit 20

# Error rate over time
fields @timestamp
| filter @message like /ERROR/
| stats count() as errors by bin(5m)

# Latency p99 (for structured logs with a duration field)
fields @timestamp, duration
| filter ispresent(duration)
| stats pct(duration, 99) as p99 by bin(1h)
```

---

## Epoch time helpers

```bash
# 1 hour ago (seconds)
date -d '1 hour ago' +%s

# 1 hour ago (milliseconds — needed for filter-log-events)
date -d '1 hour ago' +%s000

# Specific time to epoch
date -d '2026-05-04 14:00:00' +%s
```

On macOS use `date -v-1H +%s` instead of `date -d`.

---

## Common log group patterns

| Pattern | Service type |
|---------|-------------|
| `/aws/lambda/<env>-<service>` | Lambda functions |
| `/ecs/<cluster>/<service>` | ECS tasks |
| `/aws/apigateway/<api-id>` | API Gateway access logs |
| `/aws/rds/instance/<id>/error` | RDS error logs |
| `<service>-<env>` | Custom application groups |

---

## IAM permissions required

| Action | Permission |
|--------|-----------|
| List log groups | `logs:DescribeLogGroups` |
| List log streams | `logs:DescribeLogStreams` |
| Fetch events | `logs:GetLogEvents` |
| Filter events | `logs:FilterLogEvents` |
| Insights queries | `logs:StartQuery`, `logs:GetQueryResults` |

If you hit `AccessDeniedException`, tell the user which permission is missing.

---

## Critical Rules

- **NEVER print or log AWS credentials** — if they appear in output, redact them immediately
- **Always use `--limit`** when fetching events — unbounded fetches on high-volume groups are slow and expensive
- **Prefer Insights for groups > 1GB** — `get-log-events` is inefficient at scale
- **Use milliseconds for `filter-log-events` time flags**, seconds for Insights

