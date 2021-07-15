# atm-dev
Development environment for ATM

# Getting Started

## Bring up/down the dev environment
```
docker compose up
docker compose down
```

## Examples
1. Fetch call rates for both the emailservice and frontend services, grouped by operation, from now,
looking back 1 second with a sliding window of 1m and step size of 1 millisecond

    ```
    curl http://localhost:16686/api/metrics/calls?service=emailservice&service=frontend&groupByOperation=true&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000 | jq .
    ```


2. Fetch P95 latencies for both the emailservice and frontend services from now,
looking back 1 second with a sliding window of 1m and step size of 1 millisecond, where the span kind is either "server" or "client".

    ```
    curl http://localhost:16686/api/metrics/latencies?service=emailservice&service=frontend?quantile=0.95&endTs=$(date +%s)000&lookback=1000&step=1&ratePer=60000&spanKind=server&spanKind=client | jq .
    ```

3. Fetch error rates for both emailservice and frontend services using default parameters.
    ```
    curl http://localhost:16686/api/metrics/errors?service=emailservice&service=frontend | jq .
    ```

4. Fetch the minimum step size supported by the underlying metrics store.
    ```
    curl http://localhost:16686/api/metrics/minstep | jq .
    ```

# HTTP API

## Query Metrics

`/api/metrics/{metric_type}?{query}`

Where (Backus-Naur form):
```
metric_type = 'latencies' | 'calls' | 'errors'

query = services , [ '&' optionalParams ]

optionalParams = param | param '&' optionalParams

param =  groupByOperation | quantile | endTs | lookback | step | ratePer | spanKinds

services = service | service '&' services
service = 'service=' strValue
  - The list of services to include in the metrics selection filter, which are logically 'OR'ed.
  - Mandatory.

quantile = 'quantile=' floatValue
  - The quantile to compute the latency 'P' value. Valid range (0,1].
  - Mandatory for 'latencies' type.

groupByOperation = 'groupByOperation=' boolValue 
boolValue = '1' | 't' | 'T' | 'true' | 'TRUE' | 'True' | 0 | 'f' | 'F' | 'false' | 'FALSE' | 'False'
  - A boolean value which will determine if the metrics query will also group by operation.
  - Optional with default: false

endTs = 'endTs=' intValue
  - The posix milliseconds timestamp of the end time range of the metrics query.
  - Optional with default: now

lookback = 'lookback=' intValue
  - The duration, in milliseconds, from endTs to look back on for metrics data points.
  - For example, if set to `3600000` (1 hour), the query would span from `endTs - 1 hour` to `endTs`.
  - Optional with default: 3600000 (1 hour).

step = 'step=' intValue
  - The duration, in milliseconds, between data points of the query results.
  - For example, if set to 5s, the results would produce a data point every 5 seconds from the `endTs - lookback` to `endTs`.
  - Optional with default: 5000 (5 seconds).

ratePer = 'ratePer=' intValue
  - The duration, in milliseconds, in which the per-second rate of change is calculated for a cumulative counter metric.
  - Optional with default: 600000 (10 minutes).

spanKinds = spanKind | spanKind '&' spanKinds
spanKind = 'spanKind=' spanKindType
spanKindType = 'unspecified' | 'internal' | 'server' | 'client' | 'producer' | 'consumer'
  - The list of spanKinds to include in the metrics selection filter, which are logically 'OR'ed.
  - Optional with default: 'server'
```


## Min Step

`/api/metrics/minstep`

Gets the min time resolution supported by the backing metrics store, in milliseconds, that can be used in the `step` parameter.
e.g. a min step of 1 means the backend can only return data points that are at least 1ms apart, not closer.

## Responses

The response data model is based on [`MetricsFamily`](https://github.com/jaegertracing/jaeger/blob/master/model/proto/metrics/openmetrics.proto#L53).

For example:
```
{
  "name": "service_call_rate",
  "type": "GAUGE",
  "help": "calls/sec, grouped by service",
  "metrics": [
    {
      "labels": [
        {
          "name": "service_name",
          "value": "emailservice"
        }
      ],
      "metricPoints": [
        {
          "gaugeValue": {
            "doubleValue": 0.005846808321083344
          },
          "timestamp": "2021-06-03T09:12:06Z"
        },
        {
          "gaugeValue": {
            "doubleValue": 0.006960443672323934
          },
          "timestamp": "2021-06-03T09:12:11Z"
        },
...
  ```
  
If the `groupByOperation=true` parameter is set, the response will include the operation name in the labels like so:
```
      "labels": [
        {
          "name": "operation",
          "value": "/SendOrderConfirmation"
        },
        {
          "name": "service_name",
          "value": "emailservice"
        }
      ],
```

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
