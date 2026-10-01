# Demo Infrastructure

This demo creates shared infrastructure components that support one or more other APD demos. Unlike feature-specific demos, the infra demo is not tied to a single use case — it provides shared platform services that other demos can depend on.  Each component is deployed and removed independently via its own job templates.

## Prerequisites

- AAP deployed in the `aap` namespace via the AAP operator
- For components deployed on OpenShift, credentials with cluster-admin access
- For components deployed on AWS, credentials with sufficient privileges

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
| [Infoblox](README-Infoblox.md) | Infoblox virtual appliance deployed in AWS for DNS/DHCP/IPAM demos |
| [Splunk](README-Splunk.md) | Splunk Enterprise deployed on OpenShift via the Splunk Operator |
