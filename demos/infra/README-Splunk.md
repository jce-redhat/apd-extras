# Splunk

Deploys a standalone Splunk Enterprise instance on OpenShift using the Splunk Operator via OLM. Used as the Splunk infrastructure for APD demos that require a running Splunk instance.

## Prerequisites

- OpenShift 4.x cluster with cluster-admin access and OLM enabled
- Access to the `certified-operators` catalog source
- Sufficient persistent storage for Splunk volumes (20Gi per volume)
- **OpenShift Credential** — cluster-admin bearer token for the target OpenShift cluster
- **AAP Credential** — required by the Deploy Splunk instance template to set the Splunk admin password to `aap_password`

## Job Templates

* **Infra | Splunk | Install Splunk Operator** — subscribes to the Splunk Operator via OLM in the `openshift-operators` namespace. Requires: `OpenShift Credential`.

* **Infra | Splunk | Deploy Splunk instance** — creates the target namespace, sets the admin password to `aap_password`, and deploys a `Standalone` CR, waits for it to reach Ready phase, and creates an OpenShift Route for Splunk Web access. Requires: `OpenShift Credential`, `AAP Credential`.

* **Infra | Splunk | Delete Splunk instance** — removes the `Standalone` CR and associated resources, then deletes the namespace. Requires: `OpenShift Credential`.

* **Infra | Splunk | Delete Splunk Operator** — removes the OLM subscription and all Splunk CRDs from the cluster. Requires: `OpenShift Credential`.

## Workflows

**Infra | Splunk | Deploy Splunk on OpenShift** orchestrates the full deployment:

1. Installs the Splunk Operator via OLM
2. Deploys a standalone Splunk instance in the target namespace (can take 5–10 minutes to reach Ready)

**Infra | Splunk | Delete Splunk from OpenShift** orchestrates full teardown:

1. Removes the Splunk instance and namespace
2. Removes the operator and all Splunk CRDs

## Configuration

All deployment parameters are hardcoded. No survey input is required except the operator channel when installing the operator.

| Parameter | Value | Description |
|---|---|---|
| Target namespace | `apd-splunk` | OpenShift namespace where Splunk is deployed |
| Splunk instance name | `s1` | Name for the `Standalone` CR and associated resources |
| Storage size | `20Gi` | Persistent volume size for Splunk `etc` and `var` volumes |
| Admin password | `aap_password` from the AAP Credential (deploy fails immediately if not set) |

## Accessing Splunk

After successful deployment:
1. The deployment output displays the Splunk Web URL (via OpenShift Route)
2. Log in with username `admin`
3. Use the same password as the AAP admin (`aap_password`)

## Notes

- The Splunk Operator runs cluster-wide from the `openshift-operators` namespace
- OpenShift's default security policies prevent Splunk pods from running without an explicit SCC grant. The deploy playbook creates a `ClusterRoleBinding` granting the `nonroot-v2` SCC to the `default` service account in the `splunk` namespace. This is required because Splunk runs as UID/GID 41812 and the operator does not set a custom `serviceAccountName` on the `StatefulSet`. The `nonroot-v2` SCC is the correct choice per the [Splunk Operator OpenShift documentation](https://splunk.github.io/splunk-operator/) for OCP 4.14+.
- The admin password is set via the global namespace Secret (`splunk-apd-splunk-secret`) created before the Standalone CR; the operator reads from this secret during initialization rather than generating a random password
- Storage uses the cluster's default storage class
