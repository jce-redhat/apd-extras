# Splunk Enterprise Deployment Demo

This demo deploys Splunk Enterprise on an AWS EC2 instance and demonstrates automated installation and configuration of enterprise log management infrastructure.

For Event-Driven Ansible integration with Splunk and ServiceNow, see the `eda_splunk_snow` demo.

## Overview

The Splunk demo creates:
- Custom AAP credential type for Splunk admin credentials
- EC2 instance running RHEL 9 with Splunk Enterprise (t3.medium minimum)
- Additional RHEL 9 EC2 instance for testing/demonstration (t3.small default)
- Splunk Enterprise installation with Free license
- Configured data receiving on port 9997
- Splunk Web interface on port 8000
- Workflow orchestration for end-to-end deployment

## Prerequisites

### Required APD Resources
This demo requires the following resources from Ansible Product Demos:
- Cloud | AWS | Create Keypair job template
- Cloud | AWS | Create VPC job template
- Cloud | AWS | Create VM job template

### AWS Requirements
- AWS account with appropriate permissions
- Sufficient EC2 quota for t3.medium instances (minimum 4GB RAM)
- Sufficient EC2 quota for one Elastic IP address
- VPC and subnet pre-created (default: aws-test-vpc, aws-test-subnet)
- SSH keypair for EC2 access

### Splunk Requirements
- **Splunk Installation packages**: This demo requires Splunk RPMs downloaded from the [Splunk downloads page](https://www.splunk.com/en_us/download.html)
  - **Option 1 (Recommended)**: Pre-stage Splunk Enterprise RPM and Splunk Universal Forwarder RPM in an S3 bucket and provide URL
  - **Option 2**: Provide direct download URL for Splunk Enterprise RPM and Splunk Universal Forwarder RPM 

### Credentials
Set the Splunk admin password before installation:
```bash
export SPLUNK_ADMIN_PASSWORD='YourSecurePassword123!'
```

Or configure in the "Splunk Admin" credential in AAP after demo installation but before running the Splunk deployment workflow.

## Installation

### Install the Demo

Run the **APD Extras | Install extra demo** job template with:
- **demo**: `splunk`

Or using ansible-navigator:
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=splunk
```

## Usage

### Option 1: Using the Workflow (Recommended)

Run the **Deploy Splunk Enterprise in AWS** workflow and provide:
- AWS region
- AWS resource owner tag
- AWS keypair public key content
- Splunk instance name (default: apd-splunk)
- Splunk instance type (default: t3.medium)
- RHEL 9 instance name (default: apd-rhel9)
- RHEL 9 instance type (default: t3.small)
- Splunk download URL

The workflow will:
1. Create AWS keypair
2. Create AWS VPC and subnet
3. Deploy Splunk Enterprise EC2 instance and RHEL 9 instance (in parallel)
4. Sync AWS inventory
5. Install Splunk Enterprise on the Splunk server, and the Splunk Universal Forwarder on the RHEL 9 client
6. Configure Splunk Enterprise
7. Configure HAProxy for secure connections to Splunk Enterprise

## Accessing Splunk

After successful installation:
1. Find the EC2 instance Elastic IP in AWS console or AAP inventory
2. Access Splunk Web: `https://<elastic-ip>/`
3. Login with:
   - **Username**: `admin`
   - **Password**: Value of `SPLUNK_ADMIN_PASSWORD` environment variable

