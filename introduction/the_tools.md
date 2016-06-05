# The Tools
Here are my tools of choice, in no particular order, roughly catergorised.

## Configuration Management
CM tools enable use to remotely and easily configure systems in a consistent, code based way.

* [Ansible](configuration_management/ansible/ansible.md);
* [Terraform Inventory](configuration_management/ansible/terrform_inventory.md); Custom, dynamic inventory based on Terraform;

## Infrastructure Management
Like CM, IM does the same thing, but for infrastructure, such as networks and VMs.

* [Packer](infrastructure_management/packer/packer.md);
* [Terraform](infrastructure_management/terraform/terraform.md);

## Continuous Integration/Deployment
With CI and CD, we're able to automatically deliver value to our network automatically, based on certain events or triggers.

* DroneCI;
* JenkinsCI;

## Monitoring
Monitoring is a must for all environments.

* Sensu;
* Redis;
* Grafana;

## Database
We have to store our data somewhere, right?

* PostgreSQL;
* Replication Manager;

## HTTPS and Load Balancing
We must serve our application as well as balance incoming requests across mulitple stateless systems.

* Nginx;
* HAProxy;

## Logging
* ElasticSearch;
* Logstash;
* Kibana;

## Analytics
Being able to gather intelligence on who is visiting your website, and from where, using what platform, and so on, allows you to adjust your approach to market to suit your demographics.

* Piwik;
