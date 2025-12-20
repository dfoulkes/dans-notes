# Connectivity Issues Between Prometheus and Thanos

## Issue

When querying in Grafana using the `Prometheus` datasource, we are only getting data from the cache.

## Diagnosis
- The connection to prometheus at: 




### Log Line
#### Command
```Bash
kubectl logs -n monitoring -l app.kubernetes.io/name=thanos-query,app.kubernetes.io/component=query --tail 20
```


#### Finding
```Bash
ts=2025-10-19T12:49:37.041582761Z caller=proxy.go:347 level=error component=proxy err="fetch series for {prometheus=\"monitoring/prometheus-operator-kube-p-prometheus\", prometheus_replica=\"prometheus-prometheus-operator-kube-p-prometheus-0\"} Addr: prometheus-operated.monitoring.svc.cluster.local:10901 LabelSets: {prometheus=\"monitoring/prometheus-operator-kube-p-prometheus\", prometheus_replica=\"prometheus-prometheus-operator-kube-p-prometheus-0\"} MinTime: 1758286147990 MaxTime: 9223372036854775807: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing: dial tcp 10.42.0.243:10901: i/o timeout\""
```

#### Explanation
The error message indicates that Thanos Query is unable to connect to the Prometheus instance. `


### Tested Solution

<procedure>
<step number="1" title="Verify Network Connectivity">
Rerun the rollout of the statefulset for Prometheus to re-establish the connection.
<code-block language="bash">
kubectl rollout restart statefulset -n monitoring prometheus-prometheus-operator-kube-p-prometheus
</code-block>
</step>
<step number="2" title="Restart Thanos Query Deployment">
Rerollout the Thanos Query deployment to refresh its connection to Prometheus.
<code-block language="bash">
kubectl rollout restart deployment -n monitoring thanos-query
</code-block>
</step>
<step number="3" title="Check Logs Again">
After restarting both Prometheus and Thanos Query, check the logs of Thanos Query again to see if the connection issue has been resolved.
<code-block language="bash">
kubectl logs -n monitoring -l app.kubernetes.io/name=thanos-query,app.kubernetes.io/component=query --tail 20
</code-block>
</step>
</procedure>_
