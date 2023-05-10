# Persisting metrics externally

## Generate metrics

Install generator

```
go install github.com/solo-io/otel-collector-scripts/metrics-generator/cmd/metrics-generator@latest
```

Run generator

```
metrics-generator -c 2 -t 2s
```

This will generate metrics under [localhost:8080/metrics](localhost:8080/metrics)

## (Optional) Run m3db storage for prometheus

Run m3db
```
mkdir m3db_data
docker run -p 7201:7201 -p 9003:9003 --network host --name m3db \
  -v $(pwd)/m3db_data:/var/lib/m3db \
  quay.io/m3/m3dbnode:latest
```

Initialize unnaggregated db
```
 curl -X POST http://localhost:7201/api/v1/database/create -d '{
  "type": "local",
  "namespaceName": "default",
  "retentionTime": "12h"
}' | jq .
```

Verify placement:
```
curl http://localhost:7201/api/v1/services/m3db/placement | jq .
```

Mark DB as ready:
```
curl -X POST http://localhost:7201/api/v1/services/m3db/namespace/ready -d '{
  "name": "default"
}' | jq .
```

## Run Prometheus

To use m3db as a backend storage use the configuration `prometheus-with-m3db.yaml`

```
docker run -d \
  --name prometheus \
  --network host \
  -v $(pwd)/prometheus-with-m3db.yaml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

Alternatively, for no backend storage use the configuration `prometheus.yaml`

```
docker run -d \
  --name prometheus \
  --network host \
  -v $(pwd)/prometheus-with-m3db.yaml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

### Cleanup

```
docker rm -f prometheus m3db
```
