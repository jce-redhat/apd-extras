# Demo: Policy as Code with Open Policy Agent

Demonstrate AAP's Policy as Code feature using Open Policy Agent (OPA) deployed on OpenShift.  The demo deploys an OPA server, loads Rego policies from the `policies/` directory, and configures AAP to evaluate policies before allowing jobs to run.

## Prerequisites

### OpenShift Requirements

An OpenShift cluster is required.  The OpenShift credential used must have sufficient privileges to create projects, deployments, services, and routes.

### APD prerequisites

The following AAP credentials must exist before running the demo:

* **OpenShift Credential**: OpenShift bearer token credential for authenticating to the OpenShift cluster
* **AAP Credential**: Red Hat Ansible Automation Platform credential for configuring AAP settings

## Demo AAP Resources

The demo installs the following AAP resources.

### Job Templates

* **OpenShift | OPA | Deploy Open Policy Agent**: Deploys OPA on OpenShift and configures AAP's Policy as Code settings to use the new OPA server.  All `.rego` files in the `policies/` directory are loaded into a ConfigMap and mounted into the OPA container.
* **OpenShift | OPA | Add Policy**: Uploads a Rego policy to OPA via the REST API.  Policies added this way are stored in memory only and will not survive an OPA pod restart.

## Included Policies

### deny_all

A simple policy that denies all job execution.  Useful for demonstrating that Policy as Code enforcement is working, or for scenarios where job execution should be prevented such as "maintenance mode".

**Query path**: `apd/deny_all`

### common_policies

A combined policy that performs the following checks:

* **Maintenance window**: Restricts job execution to a configurable time window.  Supports HH:MM format for start and end times, wrapping windows (e.g., 22:00-06:00), and timezone configuration.
* **Superuser restriction**: Prevents superuser accounts from running jobs unless explicitly allowed.

**Query path**: `apd/common_policies`

#### Configuring common_policies with extra_vars

The `common_policies` policy reads its configuration from a `policy_as_code_vars` dictionary in the job template's extra_vars.  This allows per-job-template policy configuration without interfering with playbook variables.

Set extra_vars on a job template, inventory, or organization to configure the policy:

```yaml
policy_as_code_vars:
  maintenance_window_start: "06:00"
  maintenance_window_end: "22:00"
  maintenance_window_timezone: "America/New_York"
  allow_superuser: true
```

| Variable | Description | Default |
|---|---|---|
| `maintenance_window_start` | Start of the allowed execution window (HH:MM, 24-hour) | Not set (no restriction) |
| `maintenance_window_end` | End of the allowed execution window (HH:MM, 24-hour) | Not set (no restriction) |
| `maintenance_window_timezone` | IANA timezone for the maintenance window | `UTC` |
| `allow_superuser` | Allow superuser accounts to run jobs (`true` or `"true"`) | `false` |

The maintenance window and superuser checks only apply when their respective variables are configured.  If no `policy_as_code_vars` are set, the policy allows all jobs.

## Applying Policies

Policies are applied to AAP organizations, inventories, or job templates by setting the `opa_query_path` in the AAP UI or API.  The query path follows the format `package/rule` using the Rego package and rule names.

For example, to apply the `common_policies` policy to an organization:

1. In the AAP UI, navigate to **Access -> Organizations**
2. Edit the organization
3. Under **Policy enforcement**, set the OPA query path to `apd/common_policies`

Multiple levels of policy can be applied simultaneously.  For example, an organization can use `apd/common_policies` while individual job templates override `policy_as_code_vars` in their extra_vars to set different maintenance windows.

## Giving the Demo

1. Run the **OpenShift | OPA | Deploy Open Policy Agent** job template.  Accept the defaults or customize the namespace and container image.  After the job completes, the output displays the OPA server URL and confirms that AAP's Policy as Code settings have been configured.

2. Apply the `apd/deny_all` policy to the demo organization.  Run any job template in that organization and show that it is denied with the message "'Deny all' policy is in effect".

3. Change the organization's policy to `apd/common_policies`.  Run a job template as a regular user to show that it is now allowed (since no `policy_as_code_vars` are configured, the policy allows all jobs by default).  Run the same job as a superuser such as "admin" to demonstrate that superuser execution is denied by default.

4. Add `policy_as_code_vars` to the organization or a job template's extra_vars with a maintenance window that excludes the current time.  Run the job template to show that it is denied with a message indicating the allowed window.

5. Adjust the maintenance window to include the current time and re-run the job template to show that it is now allowed.

6. Optionally demonstrate the superuser restriction by removing `allow_superuser` from the extra_vars and running a job template as a superuser account.

7. Optionally use the **OpenShift | OPA | Add Policy** job template to upload a new policy to OPA via the REST API, demonstrating that policies can be added without redeploying.
