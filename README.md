# apd-extras

Configuration-as-code repository for creating additional AAP resources for [Ansible Product Demos](https://github.com/ansible/product-demos).

## Prerequisites

You will need an existing Ansible Automation Platform deployment with the Ansible Product Demos installed.  On your local system, the `ansible-navigator` program must be installed.

## Quick Start

1. Clone this git repo and switch to the repository directory.

```
git clone https://github.com/jce-redhat/apd-extras.git
cd apd-extras/
```

2. Set the following environment variables with a valid AAP URL, user, and password or token.  The user must have admin privileges ("is\_superuser") in AAP or be an Organization Admin for the "Ansible Product Demos (APD)" organization created by Ansible Product Demos.

```
export AAP_HOSTNAME=https://aap.example.com
export AAP_USERNAME=admin
export AAP_PASSWORD=<password>
# alternately use a token instead of a password
#export AAP_TOKEN=<token>
```

3. Run the install-demo.yml playbook with ansible-navigator, setting the `demo` extra\_var to the name of the demo you want to install.  Available demos are found under the [demos/ directory](https://github.com/jce-redhat/apd-extras/tree/main/demos)

```
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=eda_simple
```
