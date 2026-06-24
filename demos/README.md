# Available demos in apd-extras

The following demos are available:

* **eda_simple**: Basic demonstration of Event Driven Ansible that uses the ansible.eda.check\_url plugin to monitor a web URL and restart the web service when it becomes unavailable
* **infoblox**: Demonstrate Ansible integration with an Infoblox appliance for automating DNS, DHCP, and IPAM functions
* **snow**: Demonstrate updating ServiceNow CMDB records and using the CMDB as a dynamic inventory source
* **splunk**: Deploy and configure Splunk Enterprise on an AWS EC2 instance for log management and security monitoring
* **eda_splunk_snow**: Event-Driven Ansible integration demo showing automated incident creation in ServiceNow when Splunk detects user account creation events (requires `snow` and `splunk`)
* **cloudtrail_monitoring**: Create AWS CloudTrail monitoring infrastructure that captures EC2 RunInstances events and forwards them to an SQS queue via EventBridge
* **eda_cloudtrail_sqs**: Event-Driven Ansible demo that monitors an SQS queue for new EC2 instance events from CloudTrail and triggers a fact scan (requires `cloudtrail_monitoring`)
