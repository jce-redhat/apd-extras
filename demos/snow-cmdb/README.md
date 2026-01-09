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

These values are used to create an AAP "SNOW" credential during the demo installation.  If these environment variables are not set, or if you add the demo using the **APD Extras | Install extra demo** job template, the "SNOW" credential will need to be manually updated with the correct authentication information before the demo workflows or job templates can be run.

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

Before you begin, log in to the AAP UI in one browser tab and the ServiceNow UI in a second browser tab.  In the SNOW GUI, you may want to have a favorite set for viewing server records (filter on `cmdb_ci_server_list.do`).

1. Set the stage by explaining what dynamic inventory plugins are, and how they can be used to add hosts to an inventory from one or more external sources of truth.  In the AAP UI, navigate to "Infrastructure -> Inventories" and click on **Ansible Product Demos Inventory**, then click on the "Sources" tab to show the **ServiceNow CMDB** inventory source.  Click on the **ServiceNow CMDB** source to view its configuration.  Explain how the inventory file points to the inventory plugin configuration file.

2. In a new browser tab, open the [now.yml inventory plugin configuration file](https://github.com/jce-redhat/apd-extras/blob/main/demos/snow-cmdb/now.yml).  Explain how all records in the cmdb\_ci\_server table are returned by default but the results can be filtered in order to limit the records returned.  Describe how the `sysparm_query` argument tells the inventory plugin to filter on hosts with an FQDN that ends with the string "apd.example.com".  Describe how various inventory groups can be created using attributes defined in the server records, such as the operating system of the server.

3. In the SNOW browser tab, navigate to the server table.  Explain that there are no records with an FQDN that ends with "apd.example.com" currently, so no hots are being returned by the inventory plugin.  In the AAP UI, navigate to "Infrastructure -> Hosts" and show that no hosts have been added from the list of servers found in the SNOW server table.

4. In the AAP UI, navigate to "Templates", then search for "SNOW" in the search field.  Click on the **ServiceNow: Create SNOW CMDB records** workflow template and then click on the workflow visualizer.  Explain that it will run a job template that adds a number of server records to the SNOW CMDB, then runs an inventory sync to pull the new records into the AAP inventory.  Launch the workflow and wait for it to complete.

5. In the AAP UI, navigate to "Infrastructure -> Hosts" and show that there are now five new hosts.  In the SNOW UI browser tab, show that there are five new hosts as well.  Click on one of the hosts in the SNOW UI and show that the "DNS Domain" ends with "apd.example.com", which is used in the dynamic inventory configuration to filter the results.ends with "apd.example.com", which is used in the dynamic inventory configuration to filter the results.

6. At this point the demo can be more free-form.  You might:
    1. Display the [variable file](https://github.com/jce-redhat/apd-extras/blob/main/demos/snow-cmdb/vars/bulk-cmdb-records.yml) used to add the hosts.

    2. Use the add/delete job templates to add or delete a single server record to SNOW, then explain that AAP won't be updated without an inventory sync.

    3. Delete the five demo server records with the **ServiceNow: Delete SNOW CMDB records** workflow.
