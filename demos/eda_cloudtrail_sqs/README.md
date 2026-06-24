# Event-Driven Ansible: CloudTrail EC2 Events via SQS

This demo uses Event-Driven Ansible (EDA) to automatically discover and gather facts on new EC2 instances by consuming CloudTrail events from an SQS queue.

## Overview

When a new EC2 instance is launched, the event flow is:
1. EC2 `RunInstances` API call is recorded by CloudTrail
2. EventBridge rule matches the event and forwards it to an SQS queue
3. EDA rulebook polls the SQS queue using the `amazon.aws.aws_sqs_queue` source plugin
4. Rulebook matches the event and extracts the instance ID, Name tag, and region from the CloudTrail event
5. EDA triggers the "EDA | EC2 Instance Fact Scan" job template, passing the extracted values as extra vars
6. The job template looks up the instance's public IP via `ec2_instance_info`, dynamically adds it as a host, waits for SSH connectivity, and gathers facts

This approach bypasses AAP inventory entirely — the playbook resolves the instance's public IP at runtime using the instance ID from the CloudTrail event, then connects directly via SSH.

## Prerequisites

### Required APD Extras Demo
This demo **requires** the `cloudtrail_monitoring` demo to be installed first. It creates the AWS infrastructure (CloudTrail trail, EventBridge rule, SQS queue) that generates the events consumed by this demo's rulebook.

### AWS Requirements
- The AWS credential used by the EDA rulebook activation must have `sqs:ReceiveMessage`, `sqs:DeleteMessage`, and `sqs:GetQueueUrl` permissions on the `apd-cloudtrail-queue` queue
- The AWS credential used by the controller job template must have `ec2:DescribeInstances` permission to look up instance details
- EC2 instances must be launched with a `Name` tag (the instance name is extracted from the CloudTrail event and used as the hostname)
- EC2 instances must have a public IP address and allow inbound SSH (port 22) from the AAP execution environment

## Installation

### 1. Install CloudTrail Monitoring Infrastructure
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=cloudtrail_monitoring
```
Then run the **AWS | CloudTrail | Create monitoring infrastructure** job template to deploy the AWS resources.

### 2. Install EDA CloudTrail SQS Demo
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=eda_cloudtrail_sqs -e apd_extras_git_repo_branch=eda_cloudtrail_sqs
```

Or use the **APD Extras | Install extra demo** job template with:
- **demo**: `eda_cloudtrail_sqs`
- **apd_extras_git_repo_branch**: `eda_cloudtrail_sqs` (if running from a feature branch)

**Note:** The EDA project branch defaults to `main`. Pass `-e apd_extras_git_repo_branch=<branch>` when installing from a non-main branch to ensure the EDA project syncs the correct rulebook content.

## Demo AAP Resources

### EDA Resources

* **Credential Type**: `Amazon Web Services` - AWS credential type for EDA with environment variable injection
* **Credential**: `AWS` - AWS access credentials for SQS queue access
* **Rulebook Activation**: `CloudTrail EC2 to Fact Scan` - Polls SQS queue for EC2 events and triggers the job template

### Controller Resources

* **Job Template**: `EDA | EC2 Instance Fact Scan` - Looks up an EC2 instance by ID, connects via public IP, and gathers facts. Supports concurrent jobs so multiple instances launched simultaneously are each scanned independently.

### Rulebook

The `cloudtrail-ec2-sqs.yml` rulebook uses the `amazon.aws.aws_sqs_queue` source plugin to poll the SQS queue. When a `RunInstances` event is received, it extracts three fields from the CloudTrail event and passes them as extra vars to the job template:

| Variable | Source | Description |
|----------|--------|-------------|
| `instance_id` | `responseElements.instancesSet.items[0].instanceId` | EC2 instance ID |
| `instance_name` | `requestParameters.tagSpecificationSet.items[0].tags[0].value` | Name tag value |
| `aws_region` | `detail.awsRegion` | AWS region where the instance was launched |

### Playbook

The `ec2-fact-scan.yml` playbook runs in two plays:

1. **Play 1** (`localhost`): Queries AWS for the instance details using `amazon.aws.ec2_instance_info`, then uses `ansible.builtin.add_host` to dynamically add the instance to the `new_ec2_instances` group with its public IP address.
2. **Play 2** (`new_ec2_instances`): Waits for the instance to become reachable via SSH (up to 5 minutes), then gathers facts using `ansible.builtin.setup`. Facts are cached in AAP when `use_fact_cache` is enabled on the job template.

## Giving the Demo

### Setup
1. Ensure CloudTrail monitoring infrastructure is deployed (from `cloudtrail_monitoring` demo)
2. Verify the "CloudTrail EC2 to Fact Scan" rulebook activation is running in AAP
3. Open two browser tabs:
   - AAP UI (Event-Driven Ansible > Rulebook Activations)
   - AAP UI (Jobs view)

### Demo Flow

1. **Show the EDA Configuration**
   - In AAP, navigate to "Event-Driven Ansible > Rulebook Activations"
   - Click on **CloudTrail EC2 to Fact Scan** activation
   - Explain that it is polling the SQS queue for new EC2 instance events
   - Show the rulebook code highlighting the `amazon.aws.aws_sqs_queue` source and the `RunInstances` condition

2. **Trigger the Event**
   - Launch a new EC2 instance with a Name tag, either through the AWS console or CLI
   - Wait about 1 minute for CloudTrail event propagation through EventBridge to SQS

3. **Show EDA Response**
   - In AAP, check the rulebook activation fire count
   - Navigate to the Rule Audit tab to see the matched event
   - Switch to the Jobs view and show the automatically triggered "EDA | EC2 Instance Fact Scan" job
   - Click into the job and show that it queried the instance by ID, resolved the public IP, and gathered facts
   - Optionally show the cached facts for the host in the AAP inventory

4. **Explain the Value**
   - Automated discovery and scanning of new infrastructure
   - No manual intervention or inventory sync required
   - Instance is scanned directly by its public IP — no dependency on dynamic inventory
   - Concurrent job support handles multiple instances launched simultaneously
   - Event-driven response within about a minute of instance creation

## Architecture

### Event Flow
```
EC2 RunInstances API call
    |
    v
CloudTrail Trail (multi-region)
    |
    v
EventBridge Rule
  (filter: source=aws.ec2, eventName=RunInstances)
    |
    v
SQS Queue (apd-cloudtrail-queue)
    |
    v
EDA Rulebook (amazon.aws.aws_sqs_queue source)
  (condition: eventName == "RunInstances")
  Extracts: instance_id, instance_name, aws_region
    |
    v
AAP Job Template: "EDA | EC2 Instance Fact Scan"
  Play 1: ec2_instance_info → add_host (public IP)
    |
    v
  Play 2: wait_for_connection → gather facts
```

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

### Job Template Fails with SSH Timeout
- The playbook waits up to 5 minutes for SSH connectivity. If the instance takes longer to boot, increase the `timeout` value in the `wait_for_connection` task
- Verify the instance has a public IP address and the security group allows inbound SSH (port 22)
- Check that the APD Machine Credential has the correct SSH key for the instance

### Job Template Fails on ec2_instance_info
- Verify the AWS credential attached to the job template has `ec2:DescribeInstances` permission
- Check that the instance ID passed from the CloudTrail event is valid

### No Events in SQS Queue
- Verify CloudTrail is logging: `aws cloudtrail get-trail-status --name apd-cloudtrail-monitoring`
- Check EventBridge rule targets: `aws events list-targets-by-rule --rule apd-cloudtrail-events`
- Ensure the EC2 instance was launched in the same region as the EventBridge rule
- CloudTrail events typically propagate within about 1 minute
