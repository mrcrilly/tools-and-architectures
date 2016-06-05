# Flow
I would like this book to be constructed and developed so it can be used in two ways:

1. A linear fashion so people can read it from cover to cover;
2. With each section being loosely coupled to allow for picking and choosing of reading material;

It's important that people are able to cherry pick the knowledge they want and have it be generic and applicable to their problem sets. It might be that MariaDB or PHP-FPM should be configured in a particular way, but we should always give the reader the means to chose what is right for them whilst also having an opinion on the matter.

In this book I want to address the specific architecture that would support virtually any monolithic application by covering the following topics:

- Networking;
- Firewalls and traffic control/flow;
- Monitoring;
- Logging;
- Security;
- Resilience;
- High availability;
- Deployment;

These are pretty subjects that all systems engineers should take seriously. I want to cover these subjects in as complete a fashion as I can, and I will do so in the above order, strictly. This means the following technologies will be studied and documented in the following order (which may change):

- Infrastructure (Networks, VMs, etc):
    + Terraform;
- Monitoring:
    + Sensu;
    + Redis;
    + Grafana;
- Logging:
    + ElasticSearch;
    + Logstash;
    + Kibana;
- Security:
    + Theory: SSH bastions and port forwarding;
    + Fail2Ban;
    + AIDE;
- Resilience & High Availability:
    + NginX;
        * PHP-FPM will also be required;
    + HAProxy;
    + Varnish;
- Deployment:
    + Theory: Continous Integration;
    + Theory: Continous Delivery;
    + DroneCI and JenkinsCI;
- Analytics:
    + Piwik;

I want to address all of above, and I will do so by providing:

- Ansible Roles and Playbooks for standing up relevant services;
- Developing Packer build files for deploying AMIs;
- Looking at Terraform and developing code you can use to deploy the correct services and networking designs;

And much more. It'll be a big job, but it'll be useful for us all.