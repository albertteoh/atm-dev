# atm-dev
Development environment for ATM

# Getting Started

## Bring up/down the dev environment
```
docker compose up
docker compose down
```

## Example 1
Fetch call rates for both the emailservice and frontend services from now,
looking back 1 second with a sliding window of 1m and step size of 1 millisecond

```
curl http://localhost:16686/api/metrics/calls/emailservice,frontend?quantile=0.95&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000 | jq .
```


## Example 2
Fetch P95 latencies for both the emailservice and frontend services from now,
looking back 1 second with a sliding window of 1m and step size of 1 millisecond

```
curl http://localhost:16686/api/metrics/latencies/emailservice,frontend?quantile=0.95&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000 | jq .
```

## Example 3
Fetch error rates for both emailservice and frontend services using default parameters.
```
curl http://localhost:16686/api/metrics/errors/emailservice,frontend | jq .
```

## Example 4
Fetch the minimum step size supported by the underlying metrics store.
```
curl http://localhost:16686/api/metrics/minstep | jq .
```

# HTTP API

## Query Metrics

`/api/metrics/{metric_type}/{service_list}?{query_string}`

Queries R.E.D. metrics.

## Explanation of Parameters

`metric_type`

Mandatory.

The type of metrics to query for, one of: `latencies`, `calls` and `errors`

`service_list`

Mandatory.

A comma-separated list of services to query metrics for.

Example: `emailservice,frontend`

`query_string`

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

`/api/metrics/minstep`

Gets the min time resolution supported by the backing metrics store, in milliseconds, that can be used in the `step` parameter.
e.g. a min step of 1 means the backend can only return data points that are at least 1ms apart, not closer.

# Disabling Metrics Querying

Simply remove/comment out the `METRICS_STORAGE_TYPE` environment variable in the [docker-compose.yml](./docker-compose.yml) file.

Then querying any metrics endpoints results in an error message:
```
$ curl http://localhost:16686/api/metrics/minstep | jq .
{
  "data": null,
  "total": 0,
  "limit": 0,
  "offset": 0,
  "errors": [
    {
      "code": 405,
      "msg": "metrics querying is currently disabled"
    }
  ]
}
```
