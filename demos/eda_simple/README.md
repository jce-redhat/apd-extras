# Demo: Monitor a web server with EDA

This is a simple Event Driven Ansible (EDA) demo that creates a web server in AWS and sets up an EDA rulebook activation to monitor the web server URL.  If the web server is down, the rulebook will run a job template that restarts the httpd service on the web server.

## APD prerequisites

Run the "APD | Multi-demo setup" job template and choose the "cloud" and "linux" options

## Demo workflow job template

The "Deploy EDA-monitored web server" workflow automates the following actions:

*  Creates an EC2 VPC and key pair using survey input for the region, public key content, and resource owner tag.
*  Creates an EC2 web server instance named "webserver-eda".
*  Installs the "httpd" package on the web server instance and starts the service.
*  Creates a rulebook activation using the check-url.yml rulebook that monitors the URL associated with the new web server instance.  The `public_dns_name` variable set by the AWS dynamic inventory script used for the URL hostname.

These resources are then used to demonstrate the rulebook activation which is monitoring the web server URL.

## Demo steps
1. In AAP, display the "Deploy EDA-monitored web server" workflow job template and walk through the individual steps.  If desired, click on the "Hosts" and "Rulebook Activations" tabs to show that the resources which will be created by the workflow do not exist yet.

2. Run the "Deploy EDA-monitored web server" workflow job template.  Explain that the survey answers will be used by the AWS-related steps in the workflow.  Choose one of the AWS regions.  For the AWS resource owner, any arbitrary name will suffice.  The AWS key pair public key content must be the SSH public key that matches the private key used by the "APD Machine Credential" credential.  If using the RHDP catalog item for the demo, use the public key associated with the RHDP-provided bastion host.

3. After the workflow completes, click on the "Install web server software" node in the workflow visualizer.  At the end of the job template output, the URL of the new web server will be displayed.  Open the URL in a browser tab and show the RHEL test page, indicating that the web server is operational.

4. In AAP, display the "Check web server URL" rulebook activation and explain how it works.  In a separate browser tab, show the [check-url.yml](https://github.com/jce-redhat/apd-extras/blob/eda_simple/rulebooks/check-url.yml) rulebook and explain the source and rules.  Show the "LINUX | Start Service" job template and playbook if desired.

5. Run the "LINUX | Stop Service" job template to stop the httpd process on the web server.  In the survey, use "webserver-eda" for the host name or pattern, and "httpd" for the service name.  After the job completes, return to the web server browser tab and refresh.  The RHEL test page should no longer be displayed and the browser should display a connection error.  **NOTE**: the rulebook activation delay is set to 60 seconds, but depending on when it was activated, EDA may restart the web server before you get a chance to switch to the browser tab and refresh.  This step may need to be run again.

6. In AAP, display the "Check web server URL" rulebook activation and show that it is running.  When the fire count increments, switch to the Rule Audit tab and show the rule that triggered.  Switch to the Jobs tab and show the job history for the "LINUX | Start Service" job that the rulebook activation runs when the URL becomes unavailable.  After the job runs, switch to the web server browser tab and refresh to show that it now available.

