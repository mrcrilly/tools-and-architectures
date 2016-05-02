# Ansible
Now owned by RedHat, Ansible is a very flexible, powerful, and fast configuration management tool. It's architecture is very simple and when combined with a few key tools, it will be the only tool you need to define your golden images for VMs.

Throughout this guide, we will use only Ansible to configure systems. No manual steps will be defined, only Ansible Playbooks and Roles.

All Playbooks and Roles will only focus on RHEL based distributions. There are many other fine Linux distributions out there, but one must be selected and I've selected CentOS. 

## Documentation
The Ansible documentation is relatively good, but does lack some details. Review [the documentation](http://docs.ansible.com/ansible/index.html) to learn the basics of Ansible if you're not familiar.

## Playbooks
A Playbook will be provided for each tool outlined in this guide. Playbooks aren't the best way of sharing Ansible code, however, so Roles will be developed for each tool.

## Roles
Unlike a Playbook, a Role is perfect for sharing code. Each tool will have an independant Role developed for it. These Roles will be able to stand the tool up and make it available for use.