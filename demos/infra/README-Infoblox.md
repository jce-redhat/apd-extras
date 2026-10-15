# Infoblox

Deploys an Infoblox vNIOS virtual appliance in AWS for use with the Infoblox demo. The appliance provides DNS, DHCP, and IPAM services that the demo playbooks manage via the Infoblox API.  A temporary license is attached to the Infoblox appliance for demo purposes, which expires after 60 days.

## Prerequisites

- AWS credential configured in AAP
- AAP Credential configured in AAP
- An existing VPC, subnet, and key pair in the target AWS region (created by the **Cloud | AWS | Create VPC** and **Cloud | AWS | Create Keypair** job templates from the APD multi-demo setup)
- Subscription to the **Infoblox NIOS for AWS v9.x (BYOL)** AMI on the AWS Marketplace:
  1. Log in to the AWS console
  2. Go to the [Infoblox seller page](https://aws.amazon.com/marketplace/seller-profile?id=f7f03970-f903-4ee2-8df8-8ecdfa4fd581)
  3. Choose "Infoblox NIOS for AWS v9.x (BYOL)" and click "View purchase options"
  4. Accept the terms ($0.00 cost) and click "Subscribe" — approval takes approximately five minutes

## Job Templates

* **Infra | Infoblox | Display available Infoblox AMIs** — lists available Infoblox AMIs in a given AWS region. Useful for confirming marketplace subscription is active and finding the correct AMI name filter. Requires: `AWS`.

* **Infra | Infoblox | Create Infoblox instance in AWS** — creates the EC2 security group, ENIs, instance, and elastic IP for the Infoblox appliance. The initial admin password is set to the `aap_password` value injected by the AAP Credential; the job fails immediately if the AAP Credential is not attached. Requires: `AWS`, `AAP Credential`.

* **Infra | Infoblox | Update Infoblox credential** — looks up the public DNS name of the Infoblox EC2 instance and writes it into the AAP Infoblox credential so demo job templates can reach it. Also updates the credential password to match `aap_password`. Requires: `AWS`, `AAP Credential`.

## Workflows

**Infra | Infoblox | Deploy Infoblox instance in AWS** orchestrates the full deployment:

1. Creates an AWS key pair and VPC in parallel (using the APD cloud job templates)
2. Creates the Infoblox EC2 instance once both prerequisites succeed
3. Syncs the AWS dynamic inventory
4. Updates the Infoblox credential with the instance's public DNS name and admin password

> **Note:** After the instance is created it can take up to 20 minutes before it responds to web or API requests. In the AWS console, check the instance system log (Actions → Monitor and troubleshooting → Get system log) for the message "Creating NIOS image ... (this can take a while)". The instance will reboot once the image is ready.

**Infra | Infoblox | Delete Infoblox instance from AWS** terminates the appliance:

1. Terminates the EC2 instance by name using the APD **Cloud | AWS | Delete VM** job template
2. Syncs the AWS dynamic inventory to remove the host

## Configuration

The Infoblox credential type and credential are created by the infra demo install with placeholder values. The **Update Infoblox credential** workflow step populates the host and password from the deployed instance and AAP Credential respectively.

| Credential field | Initial value | Updated by workflow |
|---|---|---|
| Host | `example.invalid` | Public DNS name of the EC2 instance |
| Username | `admin` | — |
| Password | `changeme` | `aap_password` from AAP Credential |
| Verify SSL | `false` | — |
