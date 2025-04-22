# Setup Log Monitoring with Loki and Grafana 

Assumtion is you have already installed Prometheus and Grafana using HELM. 


## Deploy Loki 

Create a values file 

File: `values.yaml`

```
loki:
  commonConfig:
    replication_factor: 1
  schemaConfig:
    configs:
      - from: "2024-04-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
  pattern_ingester:
      enabled: true
  limits_config:
    allow_structured_metadata: true
    volume_enabled: true
  ruler:
    enable_api: true

minio:
  enabled: true

deploymentMode: SingleBinary
loki:
  commonConfig:
    replication_factor: 1
  schemaConfig:
    configs:
      - from: "2024-04-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
  pattern_ingester:
      enabled: true
  limits_config:

singleBinary:
  replicas: 1

# Zero out replica counts of other deployment modes
backend:
  replicas: 0
read:
  replicas: 0
write:
  replicas: 0

ingester:
  replicas: 0
querier:
  replicas: 0
queryFrontend:
  replicas: 0
queryScheduler:
  replicas: 0
distributor:
  replicas: 0
compactor:
  replicas: 0
indexGateway:
  replicas: 0
bloomCompactor:
  replicas: 0
bloomGateway:
  replicas: 0
```

Install Loki as 

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update 
```

Install Loki as 

```
helm install -n monitoring  loki grafana/loki -f values.yaml
```

validate 
```
helm list -n monitoring 

kubectl get all -n monitoring 
```

Setup Grafana to connect with Loki 

  * From Data sources -> Add Data Source   
  * Add data source of type = Loki   
  * Connection URL = `http://loki.monitoring.svc.cluster.local:3100`  
  * From HTTP Headers, Add Header `X-Scope-OrgID : admin`


## Setup Promtail to as Log Forwarder 


Setup promtail using helm as  

```
helm upgrade --install promtail grafana/promtail -n monitoring \
  --set "loki.serviceName=loki" \
  --set "loki.servicePort=3100" \
  --set "config.clients[0].url=http://loki.monitoring.svc.cluster.local:3100/loki/api/v1/push" \
  --set "config.clients[0].tenant_id=admin"
```

validate : 

```
kubect get all -n monitoring 
```

Once you see everything running, 

Explore the logs using Prometheus => Drildown => Logs 

Also install the Loki dashboard for Granfa as 

Go to Grafana → Dashboards → + Import.

Use this Dashboard ID (example):

Dashboard ID: 13639 → "Loki - Logs Panel Example"

Click Load, select your Loki data source, and import.

You can find more by searching “Loki” at:
🔗 https://grafana.com/grafana/dashboards