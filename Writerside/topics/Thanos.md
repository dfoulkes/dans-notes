# Thanos

## Overview
The relationship between Thanos and Prometheus is a crucial aspect how the
home cluster manages retention of short-term quickly accessible metrics and long-term
storage of historical metrics. 

Prometheus is responsible for scraping and storing metrics for the first
30 days, while Thanos extends this capability by providing long-term storage to two years.

## How it works
Thanos contains the following components:

- Thanos Query.
- Thanos Store Gateway.
- Thanos Ruler.

### Thanos Query
Thanos Query is the component that allows users to query metrics from both Prometheus and thanos
store gateway. It provides a unified view of metrics, allowing users to access both short-term
and long-term metrics seamlessly.

Current we have three pods of Thanos Query running in the home cluster.
```bash
 thanos-query-5947b4b965-np4db                            ●  1/1   Running         0    1↓     1↓ │
 thanos-query-5947b4b965-tklwt                            ●  1/1   Running         0     1      1 │
 thanos-query-5947b4b965-z8hmr                            ●  1/1   Running         0     3      
```
<note>
pod names are subject to change. 
</note>

### Cache Configuration
We have a cache configuration for Thanos Query that is stored in a Kubernetes ConfigMap. This
```Bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: thanos-query-cache-config
  namespace: monitoring
data:
  query-cache.yml: |
    type: IN-MEMORY
    config:
      max_size: 64MB
      validity: 24h
```
<note>
This configuration sets up an in-memory cache with a maximum size of 64MB and a validity period of 24 hours. This helps to improve the performance of queries by caching frequently accessed data.
</note>


#### Thanos Query Container

```yaml
      containers:
      - name: thanos-query
        image: quay.io/thanos/thanos:v0.36.1
        args:
        - query
        - --log.level=info # Current log level
        - --log.format=logfmt # Log format
        - --grpc-address=0.0.0.0:10901 # gRPC server address
        - --http-address=0.0.0.0:9090 # HTTP server address
        - --query.replica-label=prometheus_replica # Label is used to identify Prometheus replicas
        - --query.replica-label=ruler_replica # Label is used to identify Thanos Ruler replicas
        - --store=prometheus-operated.monitoring.svc.cluster.local:10901 # Address of the Prometheus instance
        - --store=thanos-store-gateway.monitoring.svc.cluster.local:10901  # Address of the Thanos Store Gateway
        - --store=thanos-ruler-operated.monitoring.svc.cluster.local:10901 # Address of the Thanos Ruler instance
        - --query.auto-downsampling # Enables automatic downsampling of data for better performance
        - --query.partial-response # Allows partial responses in case of some store failures
        - --query.default-evaluation-interval=1m # Default evaluation interval for queries
        - --store.sd-dns-resolver=miekgdns # DNS resolver for service discovery
        - --store.response-timeout=30s # Timeout for store responses
        - --query.max-concurrent-select=4 # Maximum number of concurrent select queries
```
### Thanos Store Gateway
Thanos Store Gateway is responsible for storing and retrieving long-term metrics from object storage. It connects to the object storage backend 
in this implemntation, we're using a minio bucket running on local network TrueNAS.
and provides an interface for querying historical metrics.


#### Gateway Volume Mounts

```yaml
      volumes:
      - name: objstore-config
        secret:
          secretName: thanos-objstore-config
      - name: cache-config
        configMap:
          name: thanos-cache-config
      - name: data
        emptyDir: {}
```
- objstore-config: This volume mounts the Kubernetes secret named thanos-objstore-config, which contains the S3 bucket details required for Thanos to connect to the object storage backend.
- cache-config: the cache-config is an in memory config map (currently 128MB) used by Thanos to store temporary data that is frequently accessed, improving performance and reducing latency when querying metrics.
- data: This volume is an empty directory that Thanos Store Gateway uses for temporary storage during

<note>
The cache is currently set to 128MB, which is a balance between performance and resource usage. Depending on the workload and available resources, this value can be adjusted.
</note>
#### What is the Role of Thanos-store container?
The Thanos-store container is responsible for storing and retrieving long-term metrics from object storage. It

```Bash
      containers:
      - name: thanos-store
        image: quay.io/thanos/thanos:v0.36.1
        args:
        - store
        - --data-dir=/var/thanos/store # Directory for storing temporary data
        - --objstore.config-file=/etc/thanos/objstore.yml # Path to the object storage configuration file
        - --index-cache.config-file=/etc/cache/cache.yml # Path to the cache configuration file
        - --log.level=info # Log level
        - --log.format=logfmt # Log format
        - --grpc-address=0.0.0.0:10901 # gRPC server address
        - --http-address=0.0.0.0:10902 # HTTP server address
        - --sync-block-duration=3m    # Frequency of syncing blocks from object storage
        - --block-sync-concurrency=20  # Number of concurrent block sync operations
        - --store.grpc.series-max-concurrency=20 # Maximum number of concurrent gRPC series requests
        - --store.grpc.series-sample-limit=0 # No limit on the number of samples per series
```
### Thanos Ruler
Thanos Ruler is responsible for evaluating alerting and recording rules against the metrics stored in Thanos. It connects to both Prometheus and Thanos Store Gateway to access the required metrics.


### Other Key Components

#### SVC for Thanos Query
```yaml
apiVersion: v1
kind: Service
metadata:
  name: thanos-query
  namespace: monitoring
  labels:
    app.kubernetes.io/name: thanos-query
    app.kubernetes.io/component: query
spec:
  selector:
    app.kubernetes.io/name: thanos-query
    app.kubernetes.io/component: query
  ports:
  - name: grpc
    port: 10901
    targetPort: grpc
  - name: http
    port: 9090
    targetPort: http
  type: ClusterIP
```

## External Links
- [Thanos Documentation](https://thanos.io/tip/thanos/getting-started.md/)
- [Current config (private repo link)](https://github.com/dfoulkes/prometheus-setup)