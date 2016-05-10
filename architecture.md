# Architecture
I want to briefly talk about architecture here. This will set an understanding as to what we're trying to achieve with the various documentation and code bases we will explore.

It's important to remember that the goal of this book is to eventually discuss various aspects of architecture and which kind of designs you can adopt to deploy and deliver your products and services. However that being said, at this point in time we will cover a single architecture for a monolithic web application.

## The Monolithic Application
A monolithic application is a large, all-in-one code base made up of tightly coupled components that cannot work without each other. In such software projects it can often be difficult to separate components or replace them, but this doesn't always hold true. A well designed and maintained monolithic code base can, and often will, perform very well and be quite maintainable well into its life cycle.

The monolithic software design is also extremely common at the time of writing. In fact many experienced and well respected software engineers will suggest starting out with a large, single application before breaking it up into a micro service architecture.

In this book I want to address the specific architecture that would support virtually any monolithic application by covering the following topics:

- Networking;
- Firewalls and traffic control/flow;
- Monitoring;
- Logging;
- Security;
- Resilience;
- High availability;
- Deployment;

These are pretry critical subjects that all systems engineers should take seriously. I want to cover these subjects in as complete a fashion as I can, and I will do so in the above order, strictly. This means the following technologies will be studied and documented in the following order (which may change):

- Networking and Firewalls:
    + Terraform;
- Monitoring:
    + InfluxDB;
    + Telegraf;
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
    + HAProxy;
    + Varnish;
- Deployment:
    + Theory: Continous Integration;
    + Theory: Continous Delivery;
    + DroneCI;
    + JenkinsCI (maybe phased out);

I want to address all of above, and I will do so by providing:

- Ansible Roles and Playbooks for standing up relevant services;
- Developing Packer build files for deploying AMIs;
- Looking at Terraform and developing code you can use to deploy the correct services and networking designs;

And much more. It'll be a big job, but it'll be useful for us all.
