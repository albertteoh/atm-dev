# atm-dev
Development environment for ATM

# Getting Started
```
# Bring up the dev environment
docker compose up

# Example
curl http://localhost:16686/api/metrics/calls/emailservice,frontend?endTs=1621415884000&lookback=1000&step=1&ratePer=600000
```

# HTTP API

- Query Metrics: `/api/metrics/{metric_type}/{service_list}?{parameters}`
- Min Step: `/api/metrics/minstep`

## Query Metrics
Queries R.E.D. metrics.

### metric_type
Mandatory.

The type of metrics to query for, one of: `latencies`, `calls` and `errors`

### service_list
Mandatory.

A comma-separated list of services to query metrics for.

### parameters
Optional.

- `endTs`: The posix milliseconds timestamp of the end time range of the metrics query.
- `lookback`: The duration, in milliseconds, from endTs to look back on for metrics data points.
  For example, if set to `3600000` (1 hour), the query would span from `endTs - 1 hour` to `endTs`.
- `step`: The duration, in milliseconds, between data points of the query results.
  For example, if set to 5s, the results would produce a data point every 5 seconds from the `endTs - lookback` to `endTs`.
- `ratePer`: The duration in which the per-second rate of change is calculated for a cumulative counter metric.

## Min Step
Gets the min time resolution supported by the backing metrics store,
e.g. a min step of 10s means the backend can only return data points that are at least 10s apart, not closer.
