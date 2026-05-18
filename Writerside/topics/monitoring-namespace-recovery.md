# Monitoring Namespace Recovery

**Service:** monitoring  
**Tier:** Critical  
**Last Updated:** 2026-05-18  
**Incident Reference:** [2026-05-18 Monitoring Namespace Deletion](/home/claw/.obsidian/incidents/2026-05-18-monitoring-namespace-deletion.md)

---

## Overview

This runbook covers recovery of the complete k3s monitoring namespace after accidental deletion or corruption. The monitoring namespace contains all core observability services:

- Prometheus Operator + Prometheus
- Grafana (with persistent dashboards)
- Loki + Alloy (log collection)
- Thanos (long-term metrics storage)
- Jaeger + Cassandra (distributed tracing)
- Backstage (service catalog)
- Alertmanager
- Unpoller (UniFi metrics)
- Syslogs receiver

---

## When to Use This Runbook

**Symptoms:**
- Monitoring namespace missing or stuck in `Terminating`
- All monitoring services unavailable
- Grafana, Prometheus, or Backstage unreachable
- Alert delivery stopped

**Do NOT use this runbook for:**
- Individual service failures (use service-specific runbooks)
- Powercut recovery (use Barn Door Protocol skill instead)
- Partial namespace corruption (troubleshoot specific component)

---

## Prerequisites

- `kubectl` access to k3s cluster
- Access to Longhorn UI for PVC restoration
- MinIO S3 credentials (Loki + Thanos storage)
- UniFi Dream Machine credentials (Unpoller)
- GitHub OAuth credentials (Backstage authentication)
- GitHub PAT (Backstage catalog integration)

### Required Credentials

**MinIO S3:**
- Access Key: _(stored in `~/source/prometheus-setup/.credentials`)_
- Secret Key: _(stored in `~/source/prometheus-setup/.credentials`)_
- Endpoint: `http://192.168.55.107:9000`
- Buckets: `loki`, `longhorn-backups`

**UniFi Dream Machine:**
- URL: `https://192.168.50.1`
- User: `dan`
- Password: _(stored in `~/source/unfi-config/.credentials`)_

**Backstage GitHub OAuth:**
- Client ID: _(stored in `~/source/golden-signals/.credentials`)_
- Client Secret: _(stored in `~/source/golden-signals/.credentials`)_

**GitHub PAT (Backstage):**
- Token: _(stored in `~/source/golden-signals/.credentials`)_

---

## Recovery Steps

### 1. Force Delete Namespace (if stuck)

If namespace is stuck in `Terminating`:

```bash
kubectl get namespace monitoring -o json | \
  jq '.spec.finalizers = []' | \
  kubectl replace --raw /api/v1/namespaces/monitoring/finalize -f -
```

Wait for namespace deletion to complete:
```bash
kubectl wait --for=delete namespace/monitoring --timeout=5m || true
```

### 2. Redeploy Prometheus Operator

```bash
cd ~/source/prometheus-setup
helm repo update
helm upgrade --install prometheus-operator prometheus-community/kube-prometheus-stack \
  -f prometheus-operator/values.yaml \
  -n monitoring --create-namespace
```

**Wait for pods:**
```bash
kubectl wait --for=condition=ready pod \
  -l app=kube-prometheus-stack-operator \
  -n monitoring --timeout=5m
```

### 3. Restore Grafana from Longhorn Backup

**Manual Longhorn UI Restore:**
1. Open Longhorn UI (`https://longhorn.foulkes.cloud`)
2. Navigate to **Backup** tab
3. Find latest Grafana backup (labeled `app=grafana`)
4. Click **Restore**, name volume `grafana`
5. Wait for volume creation

**Create PV + PVC pointing to restored volume:**

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-restored-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: longhorn
  csi:
    driver: driver.longhorn.io
    fsType: ext4
    volumeHandle: grafana
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-operator-grafana
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  volumeName: grafana-restored-pv
  resources:
    requests:
      storage: 10Gi
EOF
```

**Restart Grafana to mount restored volume:**
```bash
kubectl rollout restart statefulset/prometheus-operator-grafana -n monitoring
```

**Verify restoration:**
```bash
kubectl exec prometheus-operator-grafana-0 -n monitoring -c grafana -- \
  du -sh /var/lib/grafana/grafana.db
# Should show ~14-15MB if dashboards restored
```

### 4. Deploy Loki

```bash
cd ~/source/loki_setup/loki_helm
helm repo add grafana https://grafana.github.io/helm-charts
helm upgrade --install loki grafana/loki -f values.yaml -n monitoring
```

**Create MinIO secret:**
```bash
kubectl create secret generic loki-s3-secret -n monitoring \
  --from-literal=access-key-id=<ACCESS_KEY> \
  --from-literal=secret-access-key=<SECRET_KEY>
```

**Verify Loki Compactor:**
```bash
kubectl logs -n monitoring loki-compactor-0 --tail=20
# Should show successful S3 connection
```

### 5. Deploy Alloy (Log Collection)

```bash
cd ~/source/loki_setup/alloy_helm
helm upgrade --install alloy grafana/alloy -f values.yaml -n monitoring
```

**Verify log collection:**
```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=alloy --tail=10
# Should show "opened log stream" messages
```

### 6. Deploy Syslogs Receiver

```bash
cd ~/source/loki_setup/syslogs_helm
helm upgrade --install syslogs . -n monitoring
```

### 7. Deploy Thanos Components

**Create S3 secret:**
```bash
kubectl create secret generic thanos-objstore-config -n monitoring \
  --from-literal=objstore.yml="type: S3
config:
  bucket: loki
  endpoint: 192.168.55.107:9000
  access_key: <ACCESS_KEY>
  secret_key: <SECRET_KEY>
  insecure: true"
```

**Deploy components:**
```bash
cd ~/source/prometheus-setup
kubectl apply -f thanos-query.yaml
kubectl apply -f thanos-store-gateway.yaml
kubectl apply -f thanos-compactor.yaml
```

**⚠️ Known Issue: Thanos Compactor Longhorn CSI Bug**

If Compactor gets stuck in `ContainerCreating` with error:
```
MountVolume.MountDevice failed: format of disk failed: device apparently in use
```

**Root cause:** Longhorn CSI v1.8.1 bug - fresh volumes fail to format.

**Fix:** Switch to `emptyDir` (Compactor data is ephemeral anyway):
```bash
kubectl scale deployment thanos-compactor -n monitoring --replicas=0
kubectl delete pvc thanos-compactor-data -n monitoring
kubectl patch deployment thanos-compactor -n monitoring --type=json -p='[
  {
    "op": "replace",
    "path": "/spec/template/spec/volumes/1",
    "value": {
      "name": "data",
      "emptyDir": {
        "sizeLimit": "50Gi"
      }
    }
  }
]'
kubectl scale deployment thanos-compactor -n monitoring --replicas=1
```

**Also ensure deployment strategy is Recreate (not RollingUpdate):**
```bash
kubectl patch deployment thanos-compactor -n monitoring --type=json -p='[
  {"op":"remove","path":"/spec/strategy/rollingUpdate"},
  {"op":"replace","path":"/spec/strategy/type","value":"Recreate"}
]'
```

**Verify Thanos Store Gateway:**
```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=thanos-store-gateway --tail=20
# Should show "block loaded" messages
```

### 8. Deploy Jaeger + Cassandra

```bash
cd ~/source/Jaeger
kubectl apply -f jaeger-cassandra-statefulset.yaml
kubectl wait --for=condition=ready pod/jaeger-cassandra-0 -n monitoring --timeout=10m
kubectl apply -f jaeger-deployment.yaml
```

**Restore Cassandra from Longhorn (optional):**
Follow same Longhorn restore process as Grafana. Volume should be labeled `app=jaeger`.

**Note:** If Cassandra keyspace is missing from backup, Jaeger will run but won't store traces.

### 9. Deploy Unpoller

```bash
helm repo add k8s-at-home https://k8s-at-home.com/charts/
helm upgrade --install unpoller k8s-at-home/unifi-poller \
  --set config.unifi.url=https://192.168.50.1 \
  --set config.unifi.user=dan \
  --set config.unifi.pass="<PASSWORD>" \
  -n monitoring
```

**Verify scraping:**
```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=unifi-poller --tail=10
# Should show "Connected to UniFi controller"
```

### 10. Deploy Backstage

**Create PostgreSQL secret:**
```bash
kubectl create secret generic backstage-postgresql-secret \
  --namespace monitoring \
  --from-literal=admin-password=$(openssl rand -base64 24) \
  --from-literal=backstage-password=$(openssl rand -base64 24)
```

**Create GitHub secrets:**
```bash
kubectl create secret generic backstage-github-secret \
  --namespace monitoring \
  --from-literal=GITHUB_TOKEN=<GITHUB_PAT>

kubectl create secret generic backstage-github-oauth \
  --namespace monitoring \
  --from-literal=AUTH_GITHUB_CLIENT_ID=<OAUTH_CLIENT_ID> \
  --from-literal=AUTH_GITHUB_CLIENT_SECRET=<OAUTH_CLIENT_SECRET>
```

**Deploy Backstage:**
```bash
cd ~/source/golden-signals/platform/backstage
helm repo add backstage https://backstage.github.io/charts
helm install backstage backstage/backstage \
  --namespace monitoring \
  --values values.yaml
```

**Wait for PostgreSQL + Backstage:**
```bash
kubectl wait --for=condition=ready pod \
  -l app.kubernetes.io/name=postgresql \
  -n monitoring --timeout=10m

# If Backstage pods started before PostgreSQL, restart them:
kubectl delete pod -n monitoring -l app.kubernetes.io/name=backstage
```

### 11. Configure Grafana Datasources

**Loki datasource:**
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-loki-datasource
  namespace: monitoring
  labels:
    grafana_datasource: "1"
data:
  loki-datasource.yaml: |-
    apiVersion: 1
    datasources:
    - name: Loki
      type: loki
      access: proxy
      url: http://loki-gateway.monitoring.svc.cluster.local
      jsonData:
        httpHeaderName1: 'X-Scope-OrgID'
      secureJsonData:
        httpHeaderValue1: 'fake'
EOF
```

**Restart Grafana:**
```bash
kubectl rollout restart statefulset/prometheus-operator-grafana -n monitoring
```

### 12. Configure TLS Certificates

**Grafana:**
```bash
kubectl annotate ingress prometheus-operator-grafana -n monitoring \
  cert-manager.io/cluster-issuer=letsencrypt-production
```

**Backstage:**
```bash
kubectl annotate ingress backstage -n monitoring \
  cert-manager.io/cluster-issuer=letsencrypt-production
```

**Verify certificates:**
```bash
kubectl get certificate -n monitoring
# Should show grafana-cert-production-tls and backstage-tls as Ready
```

### 13. Configure Longhorn Recurring Backups

```bash
# Grafana backup
kubectl label volume grafana -n longhorn-system app=grafana

# Cassandra backup
kubectl label volume <cassandra-volume-name> -n longhorn-system app=jaeger
```

**Verify recurring jobs:**
```bash
kubectl get recurringjob -n longhorn-system
# Should show jobs with matching labels
```

---

## Post-Recovery Verification

### Check All Pods Running

```bash
kubectl get pods -n monitoring
```

**Expected state:**
- `prometheus-operator-*` (1/1)
- `prometheus-prometheus-operator-kube-p-prometheus-0` (3/3)
- `prometheus-operator-grafana-0` (3/3)
- `loki-*` (all Running)
- `alloy-*` (2/2 on each node)
- `thanos-*` (all Running)
- `jaeger-*` (1/1)
- `jaeger-cassandra-0` (1/1)
- `unpoller-*` (1/1)
- `alertmanager-*` (Running)
- `backstage-*` (1/1)
- `backstage-postgresql-0` (1/1)

### Verify Services

```bash
# Grafana
curl -k https://grafana.foulkes.cloud

# Backstage
curl -k https://backstage.foulkes.cloud

# Thanos Query
kubectl port-forward -n monitoring svc/thanos-query 9090:9090
curl http://localhost:9090/-/healthy
```

### Verify Data Collection

**Prometheus targets:**
```bash
kubectl port-forward -n monitoring prometheus-prometheus-operator-kube-p-prometheus-0 9090:9090
# Open http://localhost:9090/targets
```

**Loki logs:**
```bash
kubectl exec prometheus-operator-grafana-0 -n monitoring -c grafana -- \
  wget -qO- --header="X-Scope-OrgID: fake" \
  "http://loki-gateway.monitoring.svc.cluster.local/loki/api/v1/labels"
# Should return labels (not empty)
```

**Thanos long-term storage:**
```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=thanos-store-gateway --tail=20
# Should show "block loaded" messages
```

---

## Common Issues

### Prometheus Pods Stuck Pending

**Symptoms:** Prometheus StatefulSet 0/3, Longhorn VolumeAttachment errors

**Fix:**
```bash
# Identify overloaded node
kubectl top nodes

# Cordon and drain
kubectl cordon <node-name>
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Wait for pods to reschedule
kubectl wait --for=condition=ready pod \
  -l app.kubernetes.io/name=prometheus \
  -n monitoring --timeout=10m

# Uncordon
kubectl uncordon <node-name>
```

### Loki "Timestamp Too Old" Errors

**Symptoms:** Alloy logs showing `timestamp too old`, no recent logs in Grafana

**Explanation:** Alloy catching up on old pod logs. Errors will stop after processing backlog.

**Action:** Wait 5-10 minutes. Fresh logs will flow once backlog cleared.

### Backstage Not Ready

**Symptoms:** Backstage pod Running but not Ready (0/1), 503 on readiness probe

**Cause:** Backstage started before PostgreSQL was ready

**Fix:**
```bash
kubectl delete pod -n monitoring -l app.kubernetes.io/name=backstage
# Pods will restart and connect to ready PostgreSQL
```

### Jaeger CrashLoopBackOff

**Symptoms:** Jaeger pod restarting, logs show "keyspace not found"

**Options:**
1. **Accept trace data loss** - Jaeger runs but doesn't store traces
2. **Recreate keyspace:**
   ```bash
   kubectl exec -n monitoring jaeger-cassandra-0 -- cqlsh -e "
   CREATE KEYSPACE jaeger_v1_dc1 WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
   "
   kubectl delete pod -n monitoring -l app=jaeger
   ```

---

## Data Loss Assessment

### Recoverable (if backups exist)
- ✅ Grafana dashboards (Longhorn backup, weekly)
- ✅ Cassandra traces (Longhorn backup, weekly)
- ✅ Long-term metrics (Thanos S3, indefinite retention)

### Non-Recoverable
- ❌ Prometheus local metrics (7-day window)
- ❌ Loki logs (no backup configured)
- ❌ Recent traces (gap between backup and incident)

---

## Prevention

### Namespace Protection

```bash
# Add deletion protection (requires admission controller)
kubectl annotate namespace monitoring \
  "kubectl.kubernetes.io/prevent-deletion=true"
```

### Longhorn Backup Coverage

Ensure all stateful volumes have recurring backups:
```bash
kubectl get volume -n longhorn-system --show-labels
kubectl get recurringjob -n longhorn-system
```

### Configuration in Git

All Helm values and manifests must be in git:
- `prometheus-setup` repo
- `loki_setup` repo
- `Jaeger` repo
- `golden-signals` repo (Backstage)

---

## Related Runbooks

- [Barn Door Protocol](/home/claw/.openclaw/workspace/skills/barn-door-protocol/SKILL.md) - Powercut recovery
- [Prometheus Recovery](./prometheus.md) - Prometheus-specific issues
- [Grafana Recovery](./grafana.md) - Grafana-specific issues

---

## Incident History

- **2026-05-18:** Complete namespace deletion during GitOps blog post preparation ([full incident report](/home/claw/.obsidian/incidents/2026-05-18-monitoring-namespace-deletion.md))
  - Duration: 1h 51min (04:24 - 06:15 UTC)
  - Data loss: 7 days Prometheus metrics, all Loki logs, 3 days Jaeger traces
  - Recovery: Grafana dashboards fully restored, services operational
