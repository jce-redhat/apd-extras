# APD Extras

Configuration-as-code repository for creating additional demo content for [Ansible Product Demos](https://github.com/ansible/product-demos).

## Prerequisites

You will need an existing Ansible Automation Platform deployment with the Ansible Product Demos installed.  On your local system, the `ansible-navigator` program must be installed.

## Adding APD Extras to your AAP deployment

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

3. Run the install-apd-extras.yml playbook with ansible-navigator.  It will use the [Ansible Product Demos EE](https://quay.io/repository/ansible-product-demos/apd-ee-25), which contains all of the required AAP configuration-as-code collections.

```
ansible-navigator run -m stdout playbooks/install-apd-extras.yml
```

## Installing an APD Extras demo

APD Extras comes with demo content that makes use of [Ansible Product Demos](https://github.com/ansible/product-demos) components.  Available demos are found under the [demos/ directory](https://github.com/jce-redhat/apd-extras/tree/main/demos).  A demo may have certain prerequisites that need to be met, so review the demo documentation before installation.

Each demo can be installed in one of two ways:

### Install from AAP

Log in to your AAP instance and run the **APD Extras | Install extra demo** job template.  A survey will prompt for the demo name, which must match one of the demo directory names found in the [demos/ directory](https://github.com/jce-redhat/apd-extras/tree/main/demos).

### Install using ansible-navigator: 

Run the install-demo.yml playbook with ansible-navigator, defining the `demo` extra variable:

```
ansible-navigator run -m stdout playbooks/install-demo.yml -e demo=eda_simple
```

The `demo` extra variable must match one of the demo directory names found in the [demos/ directory](https://github.com/jce-redhat/apd-extras/tree/main/demos).
