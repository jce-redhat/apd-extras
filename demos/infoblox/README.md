# Demo: Deploy and configure an Infoblox virtual appliance

This demo creates an Infoblox appliance instance in AWS and provides playbooks for showing how Ansible and AAP automate management of DNS, DHCP, and IPAM functions.

## Prerequisites

### Demo setup prerequisites

Before running the `install-demo.yml` playbook to install this demo, set the `INFOBLOX_PASSWORD` environment variable to the password that you want for logging in to the new Infoblox instance as the admin user:

```
export INFOBLOX_PASSWORD=<initial_admin_password>
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=infoblox
```

A new "Infoblox" credential is set up as part of the demo installation, and when the INFOBLOX\_PASSWORD environment variable is set, it will be added to the credential when it is created.  When the Infoblox instance is created, this password is added to the instance user-data, and if the password is not set then there will be no way to log in to the new Infoblox instance.

Alternately you can run the demo setup normally, edit the "Infoblox" credential, and update the password before running the demo workflow.

### AWS account prerequisites

Access to a current Infoblox AMI from the AWS marketplace is required before the Infoblox instance can be created.

* Log in to the AWS console (RHDP users: link and credentials provided in catalog item output)
* Go to the [Infoblox seller page](https://aws.amazon.com/marketplace/seller-profile?id=f7f03970-f903-4ee2-8df8-8ecdfa4fd581) on the AWS Marketplace
* Choose the "Infoblox NIOS for AWS v9.x (BYOL)" AMI and click "View purchase options"
* Review the terms (should be a $0.00 cost) and click "Subscribe".  The request needs to be processed and approved before the AMI will be available for use with this APD demo.
* The approval process takes around five minutes.  Once the request has been approved the page will refresh with a banner stating the AMI was successfully purchased.

If this step is skipped, the playbook which deploys the Infoblox instance will fail.

### APD prerequisites
Run the "APD | Multi-demo setup" job template and choose the "cloud" category in order to create the required job templates:

* Cloud | AWS | Create Keypair
* Cloud | AWS | Create VPC

## Demo workflow job template

The "Deploy Infoblox instance in AWS" workflow automates the following actions:

* Creates an EC2 VPC and key pair using survey input for the region, public key content, and resource owner tag.
* Creates an EC2 instance named "apd-infoblox" and refreshes the AAP inventory.
  * The instance is built using the prerequisite Infoblox AMI from the AWS marketplace, and has a 60-day trial license enabled.
* Updates the "Infoblox" credential with the public DNS name of the new Infoblox instance.

**NOTE** After the instance is created it can take up to 20 minutes before it will respond to web requests for login or API calls.  In the AWS console, go to the instance and click on "Actions -> Monitor and troubleshooting -> Get system log" to see the instance OS displaying the message "Creating NIOS image ... (this can take a while)".  After the image is created, the instance will reboot, after which the instance will respond to web requests and the demo can proceed.

## Giving the demo

1. In AAP, display the "Deploy Infoblox instance in AWS" workflow job template and walk through the individual steps.  Click on the "Infrastructure -> Credentials" tab and display the "Infoblox" credential, noting that it has a place holder for the Infoblox host name.  Click on the "Infrastructure -> Hosts" tab to show that the Infoblox instance doesn't yet exist.

2. Run the "Deploy Infoblox instance in AWS" workflow job template.  Explain that the survey answers will be used by the AWS-related steps in the workflow.  Choose one of the AWS regions.  For the AWS resource owner, any arbitrary name will suffice.  The AWS key pair public key content must be the SSH public key that matches the private key used by the "APD Machine Credential" credential.  If using the RHDP catalog item for the demo, use the public key associated with the RHDP-provided bastion host.

3. After the workflow completes, click on the "Infrastructure -> Hosts" tab to show that the apd-infoblox instance is now available.  Optionally explain how the Inventory Sync workflow node functions to update the AWS dynamic inventory source.  Click on the "Infrastructure -> Credentials" and display the "Infoblox" credential again, noting that the Infoblox host name has been updated with the public DNS name of the new Infoblox instance.

4. At this point, the new Infoblox instance is still initializing and can take up to 20 minutes to be ready.  The time can be used to ask for and answer any questions from participants, explain the reason for creating a custom credential type for Infoblox automation, etc.
  * As a stretch goal, the workflow could be used to create a second Infoblox instance beforehand, which could be used for continuing the demo without waiting.  The "Infoblox" credential would need to be updated after step 3 above to point to the prepared Infoblox instance before continuing.

5. In a new browser tab, connect to the HTTPS URL of the Infoblox appliance and log in as the "admin" user.  The public DNS name of the instance can be found by locatng the the apd-infoblox host in the inventory and finding the "public\_dns\_name" host variable, or looking at the host name field of the "Infoblox" credential.  The appliance uses a self-signed certificate so the browser will warn of an unsafe connection.

6. After logging in to the Infoblox, accept the EULA, and skip (cancel) the grid setup wizard.  Go to the "Data Management" tab, then click on the "DNS" sub-tab to show that only two default zones exist.

7. In AAP, run the "Infoblox | Create DNS zones and records" job template.  This will create a forward DNS zone and a reverse DNS zone based on the survey input, and create two example records in each zone.

8. In the Infoblox web console, show that the two new zones have been created.  Click the name of each one to display the example records that were also created.
