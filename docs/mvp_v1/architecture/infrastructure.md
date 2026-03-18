# Infrastructure & Networking Diagram - NEA MVP v1

This document provides visual representations of the Kubernetes infrastructure and networking topology.

**Related Documentation:**

- [MVP v1 Roadmap](../roadmap.md) - Implementation milestones
- [Component Repository Mapping](../implementation/component_repository_mapping.md) - What gets deployed
- [ESI Endpoints](../implementation/esi_endpoints.md) - External API connections

---

## High-Level Architecture

```
                 ┌──────────────────────┐
                 │  External Services   │
                 └──────────┬───────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
    ┌─────────┐       ┌─────────┐      ┌─────────┐
    │ ESI API │       │   SDE   │      │  Local  │
    │ (HTTPS) │       │  Files  │      │   Dev   │
    └────┬────┘       └────┬────┘      └────┬────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
                           ▼
      ┌──────────────────────────────────────┐
      │    Kubernetes Cluster (k3s)          │
      │                                      │
      │  ┌────────────────────────────────┐  │
      │  │      NEA Namespace             │  │
      │  │                                │  │
      │  │  ┌────────┐  ┌────────┐  ┌───┐│  │
      │  │  │MariaDB │  │  ESI   │  │SDE││  │
      │  │  │  Pod   │  │ Parser │  │Job││  │
      │  │  │ (PVC)  │  │CronJob │  │   ││  │
      │  │  └───┬────┘  └───┬────┘  └─┬─┘│  │
      │  │      │           │          │  │  │
      │  │      └───────────┴──────────┘  │  │
      │  │                 │              │  │
      │  │          ┌──────▼──────┐       │  │
      │  │          │  MariaDB    │       │  │
      │  │          │  Service    │       │  │
      │  │          │ (ClusterIP) │       │  │
      │  │          └─────────────┘       │  │
      │  └────────────────────────────────┘  │
      │                                      │
      └────────────────┬─────────────────────┘
                       │
                       │ Port Forward
                       ▼
                 ┌──────────┐
                 │  Jupyter │
                 │ Notebook │
                 │(localhost│
                 └──────────┘
```

---

## Kubernetes Namespace Layout

```
┌─────────────────────────────────────────────────┐
│               NEA Namespace                     │
├─────────────────────────────────────────────────┤
│                                                 │
│  Deployments:                                   │
│  ├─ mariadb-deployment                         │
│  │  └─ Pod: mariadb-xxxxx                     │
│  │     ├─ Container: mariadb:10.11            │
│  │     ├─ Port: 3306                          │
│  │     ├─ Volume: mariadb-pvc                 │
│  │     └─ Env: MYSQL_ROOT_PASSWORD (secret)  │
│                                                 │
│  Services:                                      │
│  ├─ mariadb-service (ClusterIP)                │
│  │  ├─ Port: 3306 → 3306                      │
│  │  └─ Selector: app=mariadb                  │
│                                                 │
│  PersistentVolumeClaims:                       │
│  ├─ mariadb-pvc (10Gi)                         │
│  │  └─ Bound to: PersistentVolume             │
│                                                 │
│  ConfigMaps:                                    │
│  ├─ nea-config                                  │
│  │  ├─ CORPORATION_ID=12345                   │
│  │  ├─ REGION_ID=10000002                     │
│  │  └─ STAGING_LOCATION_ID=60012345           │
│                                                 │
│  Secrets:                                       │
│  ├─ mariadb-secret                              │
│  │  └─ MYSQL_ROOT_PASSWORD                    │
│  ├─ esi-token-secret                            │
│  │  ├─ CLIENT_ID                              │
│  │  ├─ CLIENT_SECRET                          │
│  │  └─ ACCESS_TOKEN                           │
│                                                 │
│  CronJobs:                                      │
│  ├─ esi-corp-refresh                            │
│  │  ├─ Schedule: "*/30 * * * *" (30 min)     │
│  │  └─ Image: neweden/nea-esi-parser:latest  │
│  ├─ esi-market-refresh                          │
│  │  ├─ Schedule: "0 2 * * *" (daily 2am)     │
│  │  └─ Image: neweden/nea-esi-parser:latest  │
│                                                 │
│  Jobs:                                          │
│  ├─ sde-import (manual trigger)                 │
│  │  └─ Image: neweden/nea-sde-parser:latest  │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Network Flow - Database Connections

```
Local Development Machine
       │
       │ Port Forward:
       │ kubectl port-forward svc/mariadb-service 3306:3306
       │
       ▼
  ┌────────────┐
  │ localhost  │
  │   :3306    │
  └──────┬─────┘
         │
         │ Port Forward Tunnel
         ▼
  ┌──────────────────┐
  │ mariadb-service  │
  │  ClusterIP       │
  │  10.43.x.x:3306  │
  └──────┬───────────┘
         │
         │ Internal Cluster Network
         ▼
  ┌──────────────────┐
  │  MariaDB Pod     │
  │  Pod IP          │
  │  10.42.x.x:3306  │
  │       │          │
  │       ▼          │
  │  ┌──────────┐    │
  │  │   PVC    │    │
  │  │/var/lib/ │    │
  │  │  mysql   │    │
  │  └──────────┘    │
  └──────────────────┘


Within Cluster (Parsers → Database)

  ┌──────────────────┐
  │  ESI Parser Pod  │
  │  or SDE Parser   │
  └──────┬───────────┘
         │
         │ Connection:
         │ mariadb-service.nea.svc.cluster.local:3306
         ▼
  ┌──────────────────┐
  │ mariadb-service  │
  └──────┬───────────┘
         │
         ▼
  ┌──────────────────┐
  │  MariaDB Pod     │
  └──────────────────┘
```

---

## Network Flow - External API Calls

```
  ┌──────────────────┐
  │ ESI Parser       │
  │   CronJob        │
  │                  │
  │ 1. Read Secret   │
  │    esi-token     │
  │ 2. Read Config   │
  │    nea-config    │
  └────────┬─────────┘
           │
           │ HTTPS (Port 443)
           │ Authorization: Bearer {token}
           ▼
  ┌──────────────────┐
  │ k3s Cluster      │
  │    Egress        │
  │ (Node external   │
  │   interface)     │
  └────────┬─────────┘
           │
           │ Internet
           ▼
  ┌──────────────────────────────────┐
  │  ESI API (esi.evetech.net)       │
  │  https://esi.evetech.net/...     │
  │                                  │
  │  Endpoints:                      │
  │  - /corporations/{id}/blueprints/│
  │  - /corporations/{id}/jobs/      │
  │  - /corporations/{id}/assets/    │
  │  - /corporations/{id}/orders/    │
  │  - /markets/{region_id}/history/ │
  └────────┬─────────────────────────┘
           │
           │ JSON Response
           ▼
  ┌──────────────────┐
  │ ESI Parser Pod   │
  │                  │
  │ 3. Parse JSON    │
  │ 4. To SQL        │
  │ 5. Insert DB     │
  └────────┬─────────┘
           │
           ▼
  ┌──────────────────┐
  │ MariaDB Service  │
  └──────────────────┘
```

---

## Network Flow - SDE Import

```
  ┌──────────────────┐
  │ SDE Parser Job   │
  │                  │
  │ 1. Download SDE  │
  │    (if needed)   │
  └────────┬─────────┘
           │
           │ HTTPS
           ▼
  ┌───────────────────────────────┐
  │  CCP SDE Repository           │
  │  https://eve-static-data-...  │
  │                               │
  │  Files:                       │
  │  - typeIDs.yaml               │
  │  - blueprints.yaml            │
  │  - etc.                       │
  └────────┬──────────────────────┘
           │
           │ YAML/JSON files
           ▼
  ┌──────────────────┐
  │ SDE Parser Pod   │
  │                  │
  │ 2. Parse YAML    │
  │ 3. To SQL        │
  │ 4. Bulk insert   │
  └────────┬─────────┘
           │
           ▼
  ┌──────────────────┐
  │ MariaDB Service  │
  └──────────────────┘


Alternative: Pre-load SDE files

  ┌──────────────────┐
  │ K8s ConfigMap    │
  │   or Volume      │
  │ (SDE mounted)    │
  └────────┬─────────┘
           │
           │ Mount at /data/sde
           ▼
  ┌──────────────────┐
  │ SDE Parser Pod   │
  │ Reads /data/sde  │
  └──────────────────┘
```

---

## Storage Architecture

```
  ┌──────────────────┐
  │  StorageClass    │
  │ (local-path/nfs) │
  └────────┬─────────┘
           │
           │ Dynamic Provisioning
           ▼
  ┌──────────────────┐
  │ PersistentVolume │
  │  Size: 10Gi      │
  │  Access: RWO     │
  │  Reclaim: Retain │
  └────────┬─────────┘
           │
           │ Bound to
           ▼
  ┌──────────────────┐
  │  PVC: mariadb-pvc│
  │  Namespace: nea  │
  └────────┬─────────┘
           │
           │ Mounted at /var/lib/mysql
           ▼
  ┌──────────────────┐
  │  MariaDB Pod     │
  │                  │
  │  Volume Mount:   │
  │  /var/lib/mysql  │
  │  → mariadb-pvc   │
  │                  │
  │  Data persists   │
  │  across restarts │
  └──────────────────┘

Data Persistence:
  ✓ Pod deletion → Data persists
  ✓ Node failure → Data persists (network storage)
  ✗ Volume deletion → Data lost (manual only)
```

---

## Security Architecture

```
┌────────────────────────────────────────────┐
│          Security Boundaries               │
└────────────────────────────────────────────┘

1. Network Isolation
┌───────────────────────────────────────────┐
│ Kubernetes Network Policies (optional MVP)│
│                                           │
│ - Allow: nea pods → mariadb-service:3306  │
│ - Allow: nea pods → external HTTPS (ESI)  │
│ - Deny: all other traffic                 │
└───────────────────────────────────────────┘

2. Secret Management
┌───────────────────────────────────────────┐
│ Kubernetes Secrets (base64 encoded)       │
│                                           │
│ mariadb-secret:                           │
│   MYSQL_ROOT_PASSWORD: <base64>          │
│                                           │
│ esi-token-secret:                         │
│   CLIENT_ID: <base64>                    │
│   CLIENT_SECRET: <base64>                │
│   ACCESS_TOKEN: <base64>                 │
│                                           │
│ Mounted as environment variables in pods  │
└───────────────────────────────────────────┘

3. Database Access Control
┌───────────────────────────────────────────┐
│ MariaDB User Permissions                  │
│                                           │
│ root user:                                │
│   - Full access (schema, migrations)      │
│   - Used by parsers and local dev         │
│                                           │
│ Future: Create limited user for notebook  │
│        (read-only access)                 │
└───────────────────────────────────────────┘

4. API Token Management
┌───────────────────────────────────────────┐
│ ESI OAuth Token                           │
│                                           │
│ - Stored in k8s Secret                    │
│ - Expires after 20 minutes                │
│ - MVP: Manual refresh (paste new token)   │
│ - Future: Implement refresh token flow    │
│                                           │
│ Required Scopes:                          │
│   - esi-corporations.read_blueprints.v1   │
│   - esi-industry.read_corporation_jobs.v1 │
│   - esi-assets.read_corporation_assets.v1 │
│   - esi-markets.read_corporation_orders.v1│
└───────────────────────────────────────────┘
```

---

## Deployment Process

```
Local Development
       │
       │ 1. Write code
       │ 2. Test locally
       │ 3. Commit to git (dev)
       │
       ▼
  ┌────────────┐
  │Build Docker│
  │   Image    │
  │docker build│
  │neweden/nea-│
  └──────┬─────┘
         │
         │ 4. Push to registry
         ▼
  ┌────────────┐
  │   Docker   │
  │  Registry  │
  │ (Hub/GHCR) │
  └──────┬─────┘
         │
         │ 5. Pull image to k3s
         ▼
  ┌────────────┐
  │k3s Cluster │
  │            │
  │kubectl     │
  │apply -f k8s│
  └──────┬─────┘
         │
         │ 6. Create/update resources
         ▼
  ┌────────────┐
  │ Running    │
  │   Pods     │
  │  (updated) │
  └────────────┘

For CronJobs:
  - Image update takes effect on next scheduled run
  - Or manually trigger:
    kubectl create job --from=cronjob/...

For Deployments:
  - Rolling update automatically
  - Or: kubectl rollout restart deployment/...
```

---

## Resource Allocation (Initial Estimates)

```
┌──────────────────────────────────────────┐
│       Resource Requirements              │
└──────────────────────────────────────────┘

MariaDB Deployment:
  requests:
    cpu: 500m (0.5 core)
    memory: 1Gi
  limits:
    cpu: 2000m (2 cores)
    memory: 4Gi
  storage: 10Gi PVC

ESI Parser CronJob:
  requests:
    cpu: 200m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi

SDE Parser Job:
  requests:
    cpu: 200m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi

Total Cluster Requirements (MVP):
  - Minimum: 2 CPU cores, 4GB RAM, 15GB storage
  - Recommended: 4 CPU cores, 8GB RAM, 20GB storage
```

---

## Monitoring & Observability

```
┌──────────────────────────────────────────┐
│      Monitoring Stack (Future)           │
└──────────────────────────────────────────┘

For MVP: Manual monitoring via kubectl and SQL

  ┌──────────────────┐
  │kubectl commands  │
  ├──────────────────┤
  │ get pods -n nea  │
  │ logs <pod> -n nea│
  │ describe pod     │
  │ top pods -n nea  │
  └──────────────────┘

  ┌──────────────────┐
  │  SQL queries     │
  ├──────────────────┤
  │ SELECT * FROM    │
  │ source_sync_     │
  │ status;          │
  │                  │
  │ SELECT * FROM    │
  │ source_refresh_  │
  │ run WHERE        │
  │ status='failed'; │
  └──────────────────┘

Future additions (post-MVP):
  - Prometheus for metrics
  - Grafana for dashboards
  - Loki for log aggregation
  - AlertManager for notifications
```

---

## Backup Strategy

```
┌──────────────────────────────────────────┐
│          Backup Approach                 │
└──────────────────────────────────────────┘

For MVP: Manual backups

Database Backup:
  ┌──────────────────────┐
  │  Manual mysqldump    │
  ├──────────────────────┤
  │ kubectl exec -it     │
  │ mariadb-pod -n nea --│
  │ mysqldump -u root -p │
  │ nea > backup.sql     │
  └──────────────────────┘

Volume Backup (if using local-path):
  ┌──────────────────────┐
  │  Snapshot PVC        │
  ├──────────────────────┤
  │ Copy volume data:    │
  │ /var/lib/rancher/k3s/│
  │ storage/pvc-xxx/     │
  └──────────────────────┘

Recommended backup schedule:
  - Daily: Database dump
  - Weekly: Full volume snapshot

Recovery:
  1. Restore PVC or volume data
  2. Redeploy MariaDB pod
  3. Verify data integrity
```

---

## Networking Ports Summary

| Service | Port | Protocol | Access |
|---------|------|----------|--------|
| MariaDB Service | 3306 | TCP | ClusterIP (internal) |
| MariaDB Port Forward | 3306 | TCP | localhost (development) |
| ESI API | 443 | HTTPS | Egress (internet) |
| SDE Download | 443 | HTTPS | Egress (internet) |

---

## DNS Resolution

```
Within Cluster:

  Full DNS: mariadb-service.nea.svc.cluster.local
  Short name (same namespace): mariadb-service
  Service IP: 10.43.x.x (ClusterIP)

  Connection strings for parsers:
    Host: mariadb-service
    Port: 3306
    Database: nea
    User: root
    Password: (from secret)

  Example connection string:
    mysql://root:${MYSQL_ROOT_PASSWORD}@mariadb-service:3306/nea
```

---

## Troubleshooting Network Issues

```
Common Issues:

1. Can't connect to MariaDB from local machine
   ✓ Check: kubectl get svc -n nea
   ✓ Check: kubectl get pods -n nea
   ✓ Solution: kubectl port-forward svc/mariadb-service 3306:3306 -n nea

2. Parser can't connect to MariaDB
   ✓ Check: kubectl logs <parser-pod> -n nea
   ✓ Check: DNS resolution:
            kubectl exec -it <pod> -n nea -- nslookup mariadb-service
   ✓ Solution: Verify service selector matches deployment labels

3. ESI API calls fail
   ✓ Check: Parser logs for HTTP status codes
   ✓ Check: Token expiration (20 minute lifespan)
   ✓ Check: Network egress allowed from cluster
   ✓ Solution: Refresh OAuth token, verify internet connectivity

4. PVC not binding
   ✓ Check: kubectl get pvc -n nea
   ✓ Check: kubectl describe pvc mariadb-pvc -n nea
   ✓ Solution: Ensure StorageClass available and provisioner running
```

---

## Next Steps

1. Implement M0: Deploy MariaDB in k3s
2. Test network connectivity (port-forward)
3. Validate storage persistence (delete pod, verify data remains)
4. Document actual IP addresses and DNS names in your environment
5. Update this diagram as infrastructure evolves
