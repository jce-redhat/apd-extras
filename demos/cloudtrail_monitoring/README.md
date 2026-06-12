# Demo: AWS CloudTrail Monitoring Infrastructure

This demo creates AWS CloudTrail monitoring infrastructure that captures EC2 RunInstances events and forwards them to an SQS queue for use with Event-Driven Ansible.

For the EDA rulebook that consumes these CloudTrail events, see the `eda_cloudtrail_sqs` demo.

## Overview

The CloudTrail monitoring demo creates:
- CloudTrail trail configured for multi-region logging
- S3 bucket for CloudTrail log storage with configurable retention (default: 7 days)
- EventBridge rule to filter EC2 RunInstances events from CloudTrail
- SQS queue for event delivery to EDA
- Automated resource cleanup playbook

**Event Flow:**
```
EC2 RunInstances API call
    ↓
CloudTrail Trail (multi-region, all management events)
    ↓
CloudTrail writes to S3 bucket
    ↓
EventBridge detects CloudTrail event
    ↓
EventBridge rule filters: source=aws.ec2, eventName=RunInstances
    ↓
Matching events sent to SQS queue
    ↓
EDA rulebook consumes from queue (see eda_cloudtrail_sqs demo)
```

## Prerequisites

### AWS Requirements
The AWS credential used must have permissions for:
- **CloudTrail**: CreateTrail, UpdateTrail, DeleteTrail, PutEventSelectors
- **S3**: CreateBucket, PutBucketPolicy, PutLifecycleConfiguration, DeleteBucket, DeleteObject
- **SQS**: CreateQueue, SetQueueAttributes, DeleteQueue
- **EventBridge**: PutRule, PutTargets, RemoveTargets, DeleteRule
- **IAM**: GetCallerIdentity

### Ansible Collections
Requires the following collections:
- `amazon.aws` >= 11.0.0
- `community.aws` >= 11.0.0

## Demo AAP Resources

The demo creates the following AAP resources:

### Job Templates
- **AWS | CloudTrail | Create monitoring infrastructure**: Creates the complete CloudTrail → EventBridge → SQS monitoring stack
- **AWS | CloudTrail | Delete monitoring infrastructure**: Removes all monitoring resources

## Usage

### Deploying CloudTrail Monitoring Infrastructure

Run the **AWS | CloudTrail | Create monitoring infrastructure** job template and provide:
- **AWS region**: Select target region (us-east-1, us-east-2, us-west-1, or us-west-2)
- **CloudTrail S3 retention days**: Number of days to retain logs (default: 7, range: 1-90)
- **Resource owner**: Tag value for tracking resource ownership

The job template will create all required resources and display the SQS queue name in the output.

## Testing the Integration

1. Launch an EC2 instance in the same region where you deployed the CloudTrail infrastructure:
   ```bash
   aws ec2 run-instances \
     --image-id ami-xxxxx \
     --instance-type t2.micro \
     --region us-east-2
   ```

2. Wait 2-3 minutes for CloudTrail event propagation

3. Verify the event reached SQS:
   ```bash
   aws sqs receive-message \
     --queue-url https://sqs.us-east-2.amazonaws.com/{account}/apd-cloudtrail-queue \
     --region us-east-2
   ```

## Cleanup

### Deleting CloudTrail Monitoring Infrastructure

Run the **AWS | CloudTrail | Delete monitoring infrastructure** job template and provide:
- **AWS region**: Select the region where resources were created (must match creation region)

The playbook will:
1. Display resources to be deleted
2. Pause for 10 seconds for confirmation
3. Delete EventBridge rule and targets
4. Delete SQS queue
5. Delete CloudTrail trail
6. Delete all objects from the S3 bucket (across all regions)
7. Delete the S3 bucket

**Warning**: This permanently deletes all CloudTrail logs. Since the trail is multi-region, logs from all AWS regions will be deleted. This process may take several minutes.

## Troubleshooting

### CloudTrail Events Not Appearing in SQS

1. Verify CloudTrail is logging:
   ```bash
   aws cloudtrail get-trail-status \
     --name apd-cloudtrail-monitoring \
     --region us-east-2
   ```
   - `IsLogging` should be `true`

2. Check EventBridge rule targets:
   ```bash
   aws events list-targets-by-rule \
     --rule apd-cloudtrail-events \
     --region us-east-2
   ```

3. Verify SQS queue policy allows EventBridge to send messages

### Deletion Takes a Long Time

The multi-region CloudTrail trail creates log files in all AWS regions. The delete playbook must remove each log file individually before deleting the S3 bucket. This is expected behavior and may take 3-5 minutes depending on the number of log files.
