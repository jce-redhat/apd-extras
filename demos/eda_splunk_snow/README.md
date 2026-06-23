# Event-Driven Ansible: Splunk to ServiceNow Integration

This demo showcases Event-Driven Ansible (EDA) capabilities by automatically creating ServiceNow incidents when Splunk detects security-relevant events.

## Overview

The integration creates an automated response workflow:
1. User account created on Linux host (triggers `useradd` command)
2. Splunk Universal Forwarder monitors system logs and sends events to Splunk Enterprise
3. Splunk detects the useradd process and triggers webhook to EDA
4. EDA rulebook receives event and launches AAP job template
5. ServiceNow incident created with enriched event data from Splunk

## Prerequisites

### Required APD Extras Demos
This demo **requires** both of the following demos to be installed first:
* **splunk**: Provides Splunk Enterprise deployment and Universal Forwarder configuration
* **snow**: Provides ServiceNow credential and CMDB integration

### Required External Configuration

The following configuration changes must be made in Splunk Enterprise:

* Install the "Red Hat Event Driven Ansible Add-on For Splunk" (requires login to splunk.com)
* Add an integration for the EDA Add-on for Splunk
  * Name: eda_splunk
  * Integration type: Webhook
  * Environment: eda_demo
  * Webhook endpoint: use endpoint from the "Splunk Events" event stream in AAP
  * Webhook Auth Type: API Key in Header
  * Authentication Token: token value from the "Splunk API token" EDA credential in AAP
* Configure an alert to forward events to EDA
  * Search pattern: `index=main process=useradd UID=*`
  * Title: RHEL Useradd
  * Permissions: Shared in App
  * Alert Type: Real-time
  * Trigger alert when: Per-Result
  * Trigger Action: Ansible Action
    * Integration Type: Webhook
    * Environment: eda_demo
    * Send All Results: Plaintext

## Installation

Review the [splunk](../splunk/README.md) and [snow](../snow/README.md) installation procedures for their installation requirements.

### 1. Install Prerequisites
```bash
# Install Splunk demo first
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=splunk

# Install ServiceNow demo
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=snow
```

### 2. Install EDA Integration Demo
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=eda_splunk_snow
```

Or use the **APD Extras | Install extra demo** job template with:
- **demo**: `eda_splunk_snow`

## Demo AAP Resources

### Job Templates

* **EDA Demo | Create incident from Splunk event**: Creates a ServiceNow incident with enrichment data from Splunk events (hostname, username, event logs, timestamp, Splunk web link)
* **EDA Demo | Add user to trigger Splunk event**: Adds a user account to a Linux host using useradd command, which triggers Splunk monitoring and the EDA workflow

### EDA Resources

* **Credential**: `Splunk API token` - Authenticates Splunk webhook calls to EDA event stream
* **Event Stream**: `Splunk Events` - Receives webhook events from Splunk Enterprise
* **Rulebook Activation**: `Splunk Useradd to ServiceNow` - Active rulebook monitoring for useradd events

## Giving the Demo

### Setup
1. Ensure Splunk Enterprise is deployed and running (from `splunk` demo)
2. Ensure RHEL 9 client has Universal Forwarder configured (from `splunk` demo)
3. Verify ServiceNow instance is accessible (from `snow` demo)
4. Open three browser tabs:
   - AAP UI (Jobs view)
   - Splunk Enterprise (Search & Reporting)
   - ServiceNow (Incidents table)

### Demo Flow

1. **Show the EDA Configuration**
   - In AAP, navigate to "Event-Driven Ansible → Rulebook Activations"
   - Click on **Splunk Useradd to ServiceNow** activation
   - Explain that it's actively listening for events on the event stream
   - Show the rulebook code highlighting the event filter (`process == "useradd"`)

2. **Show the Event Stream**
   - Navigate to "Event-Driven Ansible → Event Streams"
   - Click on **Splunk Events** stream
   - Explain how Splunk sends webhook events to this stream endpoint
   - Note the authentication token requirement

3. **Trigger the Event**
   - Navigate to "Templates" and search for "EDA Demo"
   - Launch **EDA Demo | Add user to trigger Splunk event**
   - Provide survey inputs:
     - Splunk client instance name: `apd-splunk-client`
     - New user name: `demouser`
   - Explain that this will run `useradd demouser` on the RHEL client

4. **Show Event Detection**
   - In Splunk browser tab, navigate to Search & Reporting
   - Run search: `index=main process=useradd`
   - Show the event containing the user creation
   - Highlight fields: host, process, username

5. **Show EDA Response**
   - In AAP browser tab, navigate to "Jobs"
   - Show the automatically triggered job: **EDA Demo | Create incident from Splunk event**
   - Click into the job and show the extra_vars passed from the rulebook
   - Highlight enrichment data: host, username, event logs, timestamp, Splunk link

6. **Show ServiceNow Incident**
   - In ServiceNow browser tab, refresh the Incidents table
   - Show the newly created incident
   - Open the incident and highlight:
     - Category: Security
     - Affected Host
     - Event Details with raw logs
     - Splunk Web Link (clickable)

7. **Explain the Value**
   - Automated security response
   - Event correlation across systems
   - Reduced mean time to response (MTTR)
   - Audit trail in ServiceNow
   - No manual intervention required

## Architecture

### Event Flow
```
┌──────────────────────┐
│  RHEL 9 Client       │
│  useradd demouser    │
└──────────┬───────────┘
           │ syslog
           ▼
┌──────────────────────┐
│  Splunk Forwarder    │
│  Port 9997           │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Splunk Enterprise   │
│  Event Detection     │
└──────────┬───────────┘
           │ webhook
           ▼
┌──────────────────────┐
│  EDA Event Stream    │
│  (Token Auth)        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  EDA Rulebook        │
│  Filter: useradd     │
└──────────┬───────────┘
           │ trigger
           ▼
┌──────────────────────┐
│  AAP Job Template    │
│  Create Incident     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ServiceNow          │
│  New Incident        │
└──────────────────────┘
```

## Troubleshooting

### Rulebook Activation Not Triggering
- Verify event stream is receiving events (check Event Stream logs)
- Confirm Splunk webhook is configured correctly
- Check authentication token matches between Splunk and EDA credential
- Review rulebook activation logs for filter matching issues

### Job Template Fails
- Verify ServiceNow credential is configured correctly
- Check network connectivity from AAP to ServiceNow instance
- Ensure job template survey fields match rulebook extra_vars

### No Events in Splunk
- Verify Universal Forwarder is running on client
- Check forwarder outputs.conf points to correct indexer
- Confirm syslog events are being generated
- Review Splunk indexer receiving port (9997)

