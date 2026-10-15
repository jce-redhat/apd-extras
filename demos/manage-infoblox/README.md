# Demo: Infoblox DNS/DHCP/IPAM automation

This demo shows how Ansible and AAP automate management of DNS, DHCP, and IPAM functions on an Infoblox appliance.

> **Prerequisite:** The **infra** demo must be installed and the Infoblox appliance must be deployed before running this demo. See [README-Infoblox.md](../infra/README-Infoblox.md) in the infra demo for setup instructions.

## Prerequisites

- Infoblox appliance deployed and reachable (via the infra demo)
- **Infoblox** credential in AAP configured with the appliance host, username, and password

## Installation

Run the **APD Extras | Install extra demo** job template with:
- **demo**: `infoblox`

Or using ansible-navigator:
```bash
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=manage-infoblox
```

## Demo AAP Resources

### Job Templates

* **Infoblox | Create DNS zones and records** — creates a forward DNS zone and a reverse DNS zone based on survey input, along with two example records in each zone.

## Giving the demo

1. In AAP, show the **Infoblox | Create DNS zones and records** job template. In a browser, connect to the Infoblox appliance HTTPS URL and log in as the `admin` user (the appliance uses a self-signed certificate). Accept the EULA and skip the grid setup wizard. Go to the **Data Management → DNS** tab to show that only the two default zones exist.

2. Run the **Infoblox | Create DNS zones and records** job template. The survey prompts for a forward zone name and a reverse zone CIDR.

3. In the Infoblox web console, show that the two new zones have been created. Click each zone name to display the example records that were created inside it.
