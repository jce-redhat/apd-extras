# Event-Driven Ansible: CloudTrail EC2 Events via SQS

This demo uses Event-Driven Ansible (EDA) to automatically discover and run post-provisioning on new EC2 instances by consuming CloudTrail events from an SQS queue.

## Overview

When new EC2 instances are launched, the event flow is:
1. EC2 `RunInstances` API call is recorded by CloudTrail
2. EventBridge rule matches the event and forwards it to an SQS queue
3. EDA rulebook polls the SQS queue using the `amazon.aws.aws_sqs_queue` source plugin
4. Rulebook collects all `RunInstances` events within a one-minute throttle window
5. EDA triggers the **EDA | EC2 Instance Post-provisioning** job template, passing the collected events as extra vars
6. The playbook extracts instance details from the CloudTrail events, queries EC2 for current state, dynamically adds each instance as a host, and runs post-provisioning tasks

The rulebook handles both single and batched instance launches within the throttle window. Each `RunInstances` API call produces one CloudTrail event, and the `once_after` throttle groups all events received within the window. When multiple instances are launched in a single API call (e.g. `aws ec2 run-instances --count 3`), they appear as multiple items in that event's `instancesSet`. The playbook flattens all instances across all events into a single list, so any combination of individual and bulk launches is handled in one job run.

## Prerequisites

### Required APD Extras Demo
This demo **requires** the `cloudtrail_monitoring` demo to be installed first. It is used to create the AWS infrastructure (CloudTrail trail, EventBridge rule, SQS queue) that generates the events consumed by this demo's rulebook.

### AWS Requirements
- The AWS credential used by the EDA rulebook activation must have `sqs:ReceiveMessage`, `sqs:DeleteMessage`, and `sqs:GetQueueUrl` permissions on the `apd-cloudtrail-queue` queue
- The AWS credential used by the controller job template must have `ec2:DescribeInstances` permission to look up instance details
- EC2 instances must have a public IP address and allow inbound SSH (port 22) from the AAP execution environment

## Installation

### 1. Install CloudTrail Monitoring Infrastructure
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=cloudtrail_monitoring
```
Then run the **AWS | CloudTrail | Create monitoring infrastructure** job template to deploy the AWS resources.

### 2. Install EDA CloudTrail SQS Demo
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=eda_cloudtrail_sqs
```

Or use the **APD Extras | Install extra demo** job template with:
- **demo**: `eda_cloudtrail_sqs`
- **apd_extras_git_repo_branch**: `<branch_name>` (if running from a feature branch)

## Demo AAP Resources

### EDA Resources

* **Credential Type**: `Amazon Web Services` - AWS credential type for EDA with environment variable injection
* **Credential**: `AWS` - AWS access credentials for SQS queue access
* **Rulebook Activation**: `Monitor SQS for new EC2 instances` - Polls SQS queue for EC2 events and triggers the job template

### Controller Resources

* **Job Template**: `EDA | EC2 Instance Post-provisioning` - Looks up EC2 instances from CloudTrail event data, connects via public IP, and runs post-provisioning tasks. Supports concurrent jobs via `allow_simultaneous`.

### Rulebook

The `sqs-new-instances.yml` rulebook uses the `amazon.aws.aws_sqs_queue` source plugin to poll the SQS queue. When `RunInstances` events are received, the rulebook uses `once_after` throttling with a one-minute window to batch events together. Events are grouped by `event.body.detail.eventID` so that each distinct CloudTrail event is retained.

After the throttle window closes, the rulebook passes the collected events to the job template as the `instance_events` extra var. The expression handles both singular and plural event variables from the rule engine:
- When multiple events match within the window, the rule engine provides `events` (a dict keyed by group)
- When only a single event matches, the rule engine provides `event` (a single object)

### Playbook

The `ec2-post-provision.yml` playbook runs in two plays:

1. **Play 1** (`localhost`): Iterates over `instance_events` and extracts instance details (`instanceId`, `imageId`, `privateIpAddress`, `subnetId`) from each event's `responseElements.instancesSet`. Queries AWS using `amazon.aws.ec2_instance_info` for current instance state, then uses `ansible.builtin.add_host` to add each instance to the `new_ec2_instances` group with its public IP. Hostnames use the format `Name-instanceId` to prevent collisions when multiple instances share the same Name tag.
2. **Play 2** (`new_ec2_instances`): Waits for each instance to become reachable (up to 5 minutes), then gathers facts using `ansible.builtin.setup`. Facts are cached in AAP when `use_fact_cache` is enabled on the job template.

## Giving the Demo

### Setup
1. Ensure CloudTrail monitoring infrastructure is deployed (from `cloudtrail_monitoring` demo)
2. Verify the **Monitor SQS for new EC2 instances** rulebook activation is running in AAP
3. Open two browser tabs:
   - AAP UI (Event-Driven Ansible > Rulebook Activations)
   - AAP UI (Jobs view)

### Demo Flow

1. **Show the EDA Configuration**
   - In AAP, navigate to "Event-Driven Ansible > Rulebook Activations"
   - Click on **Monitor SQS for new EC2 instances** activation
   - Explain that it is polling the SQS queue for new EC2 instance events
   - Show the rulebook code highlighting the `amazon.aws.aws_sqs_queue` source, the `RunInstances` condition, and the `once_after` throttle

2. **Trigger the Event**
   - Launch one or more EC2 instances with a Name tag
   - Wait about a minute for CloudTrail event propagation through EventBridge to SQS plus the one-minute throttle window

3. **Show EDA Response**
   - In AAP, check the rulebook activation fire count
   - Navigate to the Rule Audit tab to see the matched events
   - Switch to the Jobs view and show the automatically triggered **EDA | EC2 Instance Post-provisioning** job
   - Click into the job and show that it discovered all launched instances, resolved their public IPs, and ran post-provisioning

4. **Explain the Value**
   - Automated discovery and post-provisioning of new infrastructure
   - No manual intervention or inventory sync required
   - Instances are resolved by public IP at runtime — no dependency on dynamic inventory
   - Event throttling batches multiple launches into a single job run
   - Handles both individual launches and bulk launches (`--count`) in the same workflow
   - Event-driven response within about two minutes of instance creation

## Troubleshooting

### Rulebook Activation Not Triggering
- Verify the `cloudtrail_monitoring` infrastructure is deployed and CloudTrail is logging
- Check that the SQS queue `apd-cloudtrail-queue` exists and has messages (use `aws sqs get-queue-attributes`)
- Confirm the AWS credential in EDA has permissions to read from the SQS queue
- Review rulebook activation logs for connection or authentication errors

### EDA Activation Running Old Rulebook Content
EDA activations cache the rulebook content at creation time. Restarting the activation or re-syncing the EDA project does **not** update the cached content. To pick up new rulebook changes:
1. Delete the existing activation
2. Re-run `install-demo.yml` to recreate it with the updated content

### Job Template Fails with Connection Timeout
- The playbook waits up to 5 minutes for connectivity. If instances take longer to boot, increase the `timeout` value in the `wait_for_connection` task
- Verify instances have a public IP address and the security group allows inbound SSH (port 22)
- Check that the APD Machine Credential has the correct SSH key for the instances

### Job Template Fails on ec2_instance_info
- Verify the AWS credential attached to the job template has `ec2:DescribeInstances` permission
- Check that the instance IDs extracted from CloudTrail events are valid and the instances have not been terminated

### No Events in SQS Queue
- Verify CloudTrail is logging: `aws cloudtrail get-trail-status --name apd-cloudtrail-monitoring`
- Check EventBridge rule targets: `aws events list-targets-by-rule --rule apd-cloudtrail-events`
- Ensure the EC2 instance was launched in the same region as the EventBridge rule
- CloudTrail events typically propagate within about 1 minute
