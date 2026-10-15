# Grafana

Deploys a Grafana instance on OpenShift configured with example dashboards that visualize both OpenShift cluster health and Ansible Automation Platform resource metrics.

The deployment includes:
- Grafana via the community grafana-operator (v5 channel)
- OpenShift Prometheus as a datasource (via Thanos Querier)
- Two pre-built example dashboards:
  - **OpenShift Cluster Overview** — cluster-wide CPU, memory, pod, and node metrics
  - **AAP Resources** — job activity, capacity, health, and operational metrics from AAP Controller

AAP metrics are pulled from the AAP Controller's built-in Prometheus endpoint and made available to Grafana by enabling OpenShift user workload monitoring and deploying a `ServiceMonitor` in the `aap` namespace.

## Prerequisites

- OCS/ODF storage available (`ocs-external-storagecluster-ceph-rbd` storage class) for Grafana PVC
- **OpenShift Credential** — cluster-admin bearer token for the target OpenShift cluster
- **AAP Credential** — admin credentials for the AAP instance (used to create/revoke a metrics token for Prometheus)

## Job Templates

* **Infra | Grafana | Deploy Grafana on OpenShift** — deploys the full Grafana stack including operator, datasource, and example dashboards. Also enables OpenShift user workload monitoring and creates the AAP metrics `ServiceMonitor`. Re-running this job is safe (fully idempotent) and will apply any updates to dashboard definitions.

* **Infra | Grafana | Remove Grafana from OpenShift** — removes all deployed resources including the operator, dashboards, and AAP metrics configuration. Revokes the Prometheus metrics token created at deploy time.

## Configuration

All deployment parameters are hardcoded for the APD catalog environment:

| Parameter | Value |
|---|---|
| Grafana namespace | `apd-grafana` |
| AAP namespace | `aap` |
| PVC size | `1Gi` |
| Admin password | `aap_password` from the AAP Credential |

## Architecture

```
AAP Controller
  /api/controller/v2/metrics/
          │
          │  (ServiceMonitor + bearer token auth)
          ▼
  prometheus-user-workload
  (openshift-user-workload-monitoring)
          │
          ▼
  Thanos Querier
  (openshift-monitoring)
          │
          ▼
  GrafanaDatasource
  (OpenShift Prometheus)
          │
          ▼
  Grafana Dashboards
```

OpenShift cluster metrics follow the same path through Thanos but are scraped by the built-in cluster Prometheus from existing `ServiceMonitor` resources in `openshift-*` namespaces.

Key design decisions:

- **Single datasource** — dashboards use the same Thanos Querier datasource. User workload monitoring merges AAP metrics into Thanos alongside cluster metrics, so no second datasource is needed.
- **Token lifecycle** — a read-scoped gateway token is created at deploy time using admin credentials from the AAP Credential, and revoked when the Remove job runs. The token ID is stored alongside the token value in the metrics Secret so the delete playbook can clean it up. Token creation is skipped on re-runs if the Secret already exists.
- **Operator auth gate** — the deploy playbook waits for the Grafana Operator to reach `stageStatus: success` before applying dashboard CRs, ensuring updates are not silently dropped if the operator is mid-reconciliation after a pod restart.
- **Dashboard updates** — dashboard CRs are applied with server-side apply (`apply: true`), so changes to dashboard JSON (including panel removals) are correctly propagated on re-runs.

## Example Dashboards

### OpenShift Cluster Overview

Cluster-wide infrastructure metrics. All panels include an info tooltip (hover on the `i` icon) explaining the metric and its significance.

| Panel | Type | Description |
|---|---|---|
| Cluster CPU Usage | Gauge | % of CPU cores in use, averaged over 5m |
| Cluster Memory Usage | Gauge | % of RAM in use across all nodes |
| Running Pods | Stat | Pods currently in Running phase |
| Ready Nodes | Stat | Nodes with Ready condition true |
| Pods Not Running | Stat | Pods in Pending, Failed, or Unknown phase |
| Not-Ready Nodes | Stat | Total nodes minus Ready nodes |
| CPU Usage Over Time | Time series | CPU utilization trend |
| Memory Usage Over Time | Time series | Memory utilization trend |
| Cluster PVC Storage Usage | Gauge | % of provisioned PVC capacity in use across mounted volumes |
| Node /var Disk Usage | Gauge | % of /var partition used on the RHCOS node |
| Storage Usage Over Time | Time series | Trend of cluster PVC and node /var disk usage |

### AAP Resources

AAP Controller operational metrics. All panels include an info tooltip explaining the metric and its significance.

| Panel | Type | Description |
|---|---|---|
| Running Jobs | Stat | Jobs currently executing |
| Pending Jobs | Stat | Jobs submitted but not yet started |
| Job Success Rate | Stat | % of completed jobs that succeeded |
| Failed / Errored Jobs | Stat | Cumulative failed/error job count |
| Job Templates | Stat | Total job templates defined |
| Workflow Templates | Stat | Total workflow templates defined |
| Projects | Stat | Total SCM-backed projects |
| Managed Hosts | Stat | Total hosts across all inventories |
| License Hosts Used | Stat | % of license host entitlement consumed |
| DB Connections | Stat | Active PostgreSQL connections |
| Instance Capacity Used % | Bar gauge | Fork consumption % per controller node |
| Instance Remaining Capacity | Bar gauge | Remaining fork headroom per controller node |
| Running & Pending Jobs Over Time | Time series | Job queue trend |
| Job Completions Per 5m | Time series | Rate of terminal-state jobs by status |
| Launch Type Breakdown | Pie chart | Distribution of manual vs. scheduled vs. other launches |
| Capacity Utilization Over Time | Time series | Fork consumption % trend per controller node |
| Callback Event Queue Depth | Time series | Redis event backlog depth (callback receiver health) |
