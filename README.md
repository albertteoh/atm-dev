# atm-dev
Development environment for ATM

# Getting Started
```
# Bring up the dev environment
docker compose up


# Example 1: Fetch call rates for both the emailservice and frontend services from now,
# looking back 1 second with a sliding window of 1m and step size of 1 millisecond

curl http://localhost:16686/api/metrics/calls/emailservice,frontend?quantile=0.95&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000


# Example 2: Fetch P95 latencies for both the emailservice and frontend services from now,
# looking back 1 second with a sliding window of 1m and step size of 1 millisecond

curl http://localhost:16686/api/metrics/latencies/emailservice,frontend?quantile=0.95&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000


# Example 3: Fetch error rates for both emailservice and frontend services using default parameters.
curl http://localhost:16686/api/metrics/errors/emailservice,frontend
curl http://localhost:16686/api/metrics/minstep

# Teardown the dev environment
docker compose down
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

Required for `latencies`.

- `quantile`: The quantile to compute the latency "P" value. Valid numbers range from `0 - 1`, inclusive.

Optional.

- `endTs`: The posix milliseconds timestamp of the end time range of the metrics query. Default: now.
- `lookback`: The duration, in milliseconds, from endTs to look back on for metrics data points.
  For example, if set to `3600000` (1 hour), the query would span from `endTs - 1 hour` to `endTs`. Default: 1h.
- `step`: The duration, in milliseconds, between data points of the query results.
  For example, if set to 5s, the results would produce a data point every 5 seconds from the `endTs - lookback` to `endTs`. Default: 5s.
- `ratePer`: The duration in which the per-second rate of change is calculated for a cumulative counter metric. Default: 10m.

## Min Step
Gets the min time resolution supported by the backing metrics store, in milliseconds, that can be used in the `step` parameter.
e.g. a min step of 1000000 means the backend can only return data points that are at least 1ms apart, not closer.
