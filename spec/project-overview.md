# Project Overview

The apd-extras project provides supplemental demo content for the [Ansible Product Demos](https://github.com/ansible/product-demos/) (APD) project.  It consists of Ansible Automation Platform (AAP) configuration as code for creating AAP resources.  It uses the [infra.aap_configuration](https://github.com/redhat-cop/infra.aap_configuration/) collection to manage these resources, and takes advantage of resources that have been defined in the APD repository.  apd-extras is meant to be a standalone repository for proving that a demo will work within the APD structure, as a first step towards merging the demo into the main APD repository.  Content in the repository may be dependent on resources created by the APD project.

When creating demo content in the apd-extras repository:

* Use Inventories, Credentials, Projects, Job Templates, and other AAP resources created by the main APD repo rather than creating new versions of the same.  See existing demo content for examples.
* Each demo gets its own subdirectory under demos/.  Ensure each demo has a setup.yml file for creating AAP resources with config as code, a skeleton README.md file, and any playbooks associated with the demo.

