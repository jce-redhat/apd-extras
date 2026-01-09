# Demo: Manage CMDB records in ServiceNow

Demonstrate adding and removing CMDB server records from ServiceNow, including asset relationships such as assigning servers to racks, or racks to computer rooms.  A dynamic inventory plugin configuration is provided for updating the AAP inventory after records are added or removed.

## Prerequisites

A pre-existing ServiceNow instance is required for this demo, with a user having sufficient privileges to add and remove configuration items in the CMDB.  Some knowledge of ServiceNow (CMDB concepts, navigating the UI) is assumed.

### Demo setup prerequisites

Before running the `install-demo.yml` playbook to install this demo, set the following environment variables for authenticating to your ServiceNow instance:

```
export SN_HOST=https://<your-snow-instance>
export SN_USERNAME=<user>
export SN_PASSWORD=<password>
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=snow-cmdb
```

These values are used to create an AAP "SNOW" credential during the demo installation.  If these environment variables are not set, the "SNOW" credential will need to be manually updated with the correct authentication information before the demo workflows or job templates can be run.

## Demo AAP resources

The demo installs the following AAP resources.

### Credential Types

* **ServiceNow User/Password**: A custom credential type for authentication information used to log in to ServiceNow with a username and password.

### Credentials

* **SNOW**: Credential for authenticating to a ServiceNow instance, using values from the *SN\_** environment variables set during the demo installation.

### Inventory Sources

* **ServiceNow CMDB**: Uses the servicenow.itsm.now dynamic inventory plugin to create AAP inventory hosts from the ServiceNow CMDB.

### Job Templates

* **SNOW | Bulk add demo CMDB server records**: Creates a number of pre-defined server, rack, and computer room records.  The record definitions can be found in [vars/bulk-cmdb-records.yml](vars/bulk-cmdb-records.yml).
* **SNOW | Bulk delete demo CMDB server records**: Deletes the asset records created by the bulk add job template.
* **SNOW | Create a server record in the CMDB**: Creates a single CMDB server record, using a survey to request the server FQDN and optional information.
* **SNOW | Delete a server record from the CMDB**: Deletes a single CMDB server record.

### Workflows

* **ServiceNow: Create SNOW CMDB records**: Calls the bulk add job template to create the demo CMDB records, then does an inventory sync to pull in the new hosts from the CMDB.
* **ServiceNow: Delete SNOW CMDB records**: Deletes the pre-configured records created by the previous workflow, then does an inventory sync to remove the hosts from the AAP inventory.

## Giving the demo
