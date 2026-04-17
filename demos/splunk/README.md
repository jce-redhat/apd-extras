# Splunk Enterprise Demo

This demo deploys Splunk Enterprise on an AWS EC2 instance and demonstrates automated installation and configuration of enterprise log management infrastructure.

## Overview

The Splunk demo creates:
- Custom AAP credential type for Splunk admin credentials
- EC2 instance running Amazon Linux 2023 (t3.medium minimum)
- Splunk Enterprise installation with Free license
- Configured data receiving on port 9997
- Splunk Web interface on port 8000
- Workflow orchestration for end-to-end deployment

## Prerequisites

### Required APD Resources
This demo requires the following resources from Ansible Product Demos:
- "Ansible Product Demos (APD)" organization
- "Ansible Product Demos Inventory" with AWS dynamic inventory configured
- AWS credential configured in AAP
- AAP Credential for API access
- Cloud | AWS | Create Keypair job template
- Cloud | AWS | Create VPC job template
- AWS Inventory sync job template
- SUBMIT FEEDBACK job template

### AWS Requirements
- AWS account with appropriate permissions
- Sufficient EC2 quota for t3.medium instances (minimum 4GB RAM)
- VPC and subnet pre-created (default: aws-test-vpc, aws-test-subnet)
- SSH keypair for EC2 access

### Splunk Requirements
- **Splunk Binary**: This demo requires a Splunk Enterprise installation package
  - **Option 1 (Recommended)**: Pre-stage Splunk Free .tgz in an S3 bucket and provide URL
  - **Option 2**: Provide direct download URL from Splunk.com (requires authentication)
  - **Option 3**: Modify `install-splunk.yml` to download from authenticated source

### Credentials
Set the Splunk admin password before installation:
```bash
export SPLUNK_ADMIN_PASSWORD='YourSecurePassword123!'
```

Or configure in the "Splunk Admin" credential in AAP after demo installation.

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

The workflow will:
1. Create AWS keypair
2. Create AWS VPC and subnet
3. Deploy Splunk EC2 instance
4. Sync AWS inventory
5. Install Splunk Enterprise
6. Configure Splunk settings

### Option 2: Using Individual Job Templates

#### 1. Deploy EC2 Instance
Run **Splunk | AWS | Deploy Splunk Enterprise EC2 instance**:
- AWS region
- VPC name
- Subnet name
- Keypair name
- Instance name
- Instance type

#### 2. Sync AWS Inventory
Run **AWS Inventory** to discover the new instance

#### 3. Install Splunk
Run **Splunk | Install Splunk Enterprise**:
- Splunk instance name (must match EC2 instance name tag)
- Splunk download URL (optional if using default)

#### 4. Configure Splunk
Run **Splunk | Configure Splunk settings**:
- Splunk instance name
- Enable Splunk Web (true/false)

## Accessing Splunk

After successful installation:
1. Find the EC2 instance public IP in AWS console or AAP inventory
2. Access Splunk Web: `http://<public-ip>:8000`
3. Login with:
   - **Username**: `admin`
   - **Password**: Value of `SPLUNK_ADMIN_PASSWORD` environment variable

## Architecture

### EC2 Instance Specifications
- **OS**: Red Hat Enterprise Linux 9
- **Instance Type**: t3.medium (default) - 2 vCPU, 4GB RAM
- **Storage**: 120GB gp3 EBS volume
- **Security Group**: Ports 22, 8000, 8089, 9997 open

### Splunk Configuration
- **Installation Path**: `/opt/splunk`
- **User**: `splunk`
- **License**: Splunk Free (500MB/day indexing limit)
- **Splunk Web**: Port 8000
- **Management Port**: Port 8089
- **Receiving Port**: Port 9997 (for forwarders)

## Cost Considerations

**Warning**: This demo creates resources with ongoing costs:
- t3.medium EC2 instance: ~$0.0416/hour (~$30/month if running continuously)
- RHEL licensing: Additional hourly charges for Red Hat subscription
- EBS storage: ~$12/month for 120GB gp3 volume

**Recommendation**: Terminate the EC2 instance when not in use to avoid ongoing charges.

## Extending the Demo

### Adding Forwarders
Create additional job templates to:
- Deploy Universal Forwarders to application servers
- Configure forwarding to the Splunk indexer
- Set up data inputs and parsing

### Clustering
Expand to multi-node deployment:
- Search head cluster
- Indexer cluster
- Deployment server

### Apps and Add-ons
Install Splunk apps:
- Security apps (ES, SOAR)
- IT Operations apps (ITSI)
- Data inputs (AWS, Linux, Windows)

## Troubleshooting

### Splunk Installation Fails
- Verify `SPLUNK_ADMIN_PASSWORD` is set and meets complexity requirements
- Check Splunk download URL is accessible from EC2 instance
- Ensure sufficient disk space (120GB available)

### Cannot Access Splunk Web
- Verify security group allows port 8000 from your IP
- Check Splunk service is running: `systemctl status splunk`
- Verify public IP address is correct

### Inventory Sync Doesn't Find Instance
- Wait 30-60 seconds after EC2 deployment for instance to be fully tagged
- Verify instance has correct tags (Name, owner, splunk_role)
- Re-run AWS Inventory sync

## References

- [Splunk Enterprise Documentation](https://docs.splunk.com/Documentation/Splunk)
- [Splunk Free License](https://www.splunk.com/en_us/products/splunk-enterprise/free-vs-enterprise.html)
- [ansible-role-for-splunk](https://github.com/splunk/ansible-role-for-splunk)
