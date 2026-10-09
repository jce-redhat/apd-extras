# Demo Infrastructure

This demo installs shared infrastructure components on OpenShift that support one or more other APD demos. Unlike feature-specific demos, the infra demo is not tied to a single use case — it provides shared platform services that other demos can depend on.

Running the **APD Extras | Install extra demo** job template with `demo=infra` creates the AAP job templates for all components listed below. Each component is then deployed and removed independently via its own job templates.

## Prerequisites

- OpenShift cluster with cluster-admin access
- AAP deployed in the `aap` namespace via the AAP operator

## Installation

Run the **APD Extras | Install extra demo** job template with:
- **demo**: `infra`

Or using ansible-navigator:
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=infra
```

## Components

| Component | Description |
|---|---|
| [Grafana](README-Grafana.md) | Grafana dashboards for OpenShift cluster and AAP resource metrics |
