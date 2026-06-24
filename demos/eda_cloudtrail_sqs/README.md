# Event-Driven Ansible: CloudTrail EC2 Events via SQS

This demo uses Event-Driven Ansible (EDA) to automatically run a fact scan on new EC2 instances by consuming CloudTrail events from an SQS queue.

## Overview

When a new EC2 instance is launched, the event flow is:
1. EC2 `RunInstances` API call is recorded by CloudTrail
2. EventBridge rule matches the event and forwards it to an SQS queue
3. EDA rulebook polls the SQS queue using the `amazon.aws.aws_sqs_queue` source plugin
4. Rulebook matches the event and triggers the "EDA | CloudTrail EC2 Fact Scan" workflow
5. The workflow syncs the AWS dynamic inventory (so the new instance is available as a host)
6. The workflow runs a fact scan against the new instance

## Prerequisites

### Required APD Extras Demo
This demo **requires** the `cloudtrail_monitoring` demo to be installed first. It creates the AWS infrastructure (CloudTrail trail, EventBridge rule, SQS queue) that generates the events consumed by this demo's rulebook.

### Required APD Resources
- The **LINUX | Fact Scan** job template must exist in your AAP deployment. This is provided by [Ansible Product Demos](https://github.com/ansible/product-demos) and should already be available if APD is installed.
- An **AWS Inventory** inventory source must exist in the AAP inventory. The demo's workflow syncs this source before running the fact scan so that newly launched instances are available as hosts.

### AWS Requirements
- The AWS credential used by the EDA rulebook activation must have `sqs:ReceiveMessage`, `sqs:DeleteMessage`, and `sqs:GetQueueUrl` permissions on the `apd-cloudtrail-queue` queue
- EC2 instances must be launched with a `Name` tag (the instance name is extracted from the CloudTrail event and used as the `_hosts` limit for the fact scan)

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

## Demo AAP Resources

### EDA Resources

* **Credential Type**: `Amazon Web Services` - AWS credential type for EDA with environment variable injection
* **Credential**: `AWS` - AWS access credentials for SQS queue access
* **Rulebook Activation**: `CloudTrail EC2 to Fact Scan` - Polls SQS queue for EC2 events and triggers the workflow

### Controller Resources

* **Workflow Template**: `EDA | CloudTrail EC2 Fact Scan` - Two-node workflow that syncs the AWS inventory then runs a fact scan against the new instance

### Rulebook

The `cloudtrail-ec2-sqs.yml` rulebook uses the `amazon.aws.aws_sqs_queue` source plugin to poll the SQS queue. When a `RunInstances` event is received, it extracts the EC2 instance name from the CloudTrail event and passes it as the `_hosts` variable to the "EDA | CloudTrail EC2 Fact Scan" workflow template.

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
   - Launch a new EC2 instance with a Name tag, either through the AWS console or by running an APD workflow that creates an EC2 instance
   - Wait 2-3 minutes for CloudTrail event propagation through EventBridge to SQS

3. **Show EDA Response**
   - In AAP, check the rulebook activation fire count
   - Navigate to the Rule Audit tab to see the matched event
   - Switch to the Jobs view and show the automatically triggered "EDA | CloudTrail EC2 Fact Scan" workflow
   - Expand the workflow to show the two steps: inventory sync followed by fact scan
   - Click into the fact scan job and show that it targeted the new instance

4. **Explain the Value**
   - Automated discovery and scanning of new infrastructure
   - No manual intervention required
   - Consistent compliance and configuration baseline
   - Event-driven response within minutes of instance creation

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
    |
    v
AAP Workflow: "EDA | CloudTrail EC2 Fact Scan"
  Node 1: Sync AWS Inventory
    |
    v
  Node 2: "LINUX | Fact Scan"
  (limit: EC2 instance Name tag)
```

## Troubleshooting

### Rulebook Activation Not Triggering
- Verify the `cloudtrail_monitoring` infrastructure is deployed and CloudTrail is logging
- Check that the SQS queue `apd-cloudtrail-queue` exists and has messages (use `aws sqs get-queue-attributes`)
- Confirm the AWS credential in EDA has permissions to read from the SQS queue
- Review rulebook activation logs for connection or authentication errors

### Workflow or Fact Scan Job Fails
- Verify the EC2 instance was launched with a `Name` tag
- Check that the "EDA | CloudTrail EC2 Fact Scan" workflow template exists
- If the inventory sync node fails, verify the AWS Inventory source is configured and has valid credentials
- If the fact scan node fails, ensure the instance name matches a host in the AAP inventory and that the "LINUX | Fact Scan" job template exists
- If launching multiple instances, ensure each has a unique `Name` tag — instances sharing the same Name tag will collide in the dynamic inventory and only one will be scanned

### No Events in SQS Queue
- Verify CloudTrail is logging: `aws cloudtrail get-trail-status --name apd-cloudtrail-monitoring`
- Check EventBridge rule targets: `aws events list-targets-by-rule --rule apd-cloudtrail-events`
- Ensure the EC2 instance was launched in the same region as the EventBridge rule
- CloudTrail events may take 2-3 minutes to propagate
