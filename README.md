# apd-extras

Configuration-as-code repository for creating additional AAP resources for [Ansible Product Demos](https://github.com/ansible/product-demos).

## Prerequisites

An existing Ansible Automation Platform deployment with the Ansible Product Demos included.

## Quick Start

1. Set the following environment variables with a valid AAP URL, user, and password or token.  The user must have admin privileges ("is\_superuser") in AAP or be an Organization Admin for the "Ansible Product Demos (APD)" organization created by Ansible Product Demos.

```
export AAP_HOSTNAME=https://aap.example.com
export AAP_USERNAME=admin
export AAP_PASSWORD=<password>
# alternately use a token instead of a password
#export AAP_TOKEN=<token>
```
