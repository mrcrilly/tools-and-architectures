# Architecture
I want to briefly talk about architecture here. This will set an understanding as to what we're trying to achieve with the various documentation and code bases we will explore.

It's important to remember that the goal of this book is to eventually discuss various aspects of architecture and which kind of designs you can adopt to deploy and deliver your products and services. However that being said, at this point in time we will cover a single architecture for a monolithic web application.

## The Monolithic Application
A monolithic application is a large, all-in-one code base made up of tightly coupled components that cannot work without each other. In such software projects it can often be difficult to separate components or replace them, but this doesn't always hold true. A well designed and maintained monolithic code base can, and often will, perform very well and be quite maintainable well into its life cycle.

The monolithic software design is also extremely common at the time of writing. In fact many experienced and well respected software engineers will suggest starting out with a large, single application before breaking it up into a micro service architecture.

## Demonstration Application
Ideally we would have a real-world application to use for demonstation purposes, but to keep things simple, and to allow us to make changes to the application in a manner the reader can follow along with, we will use a very simple Django application that just serves responses over HTTPS. The application will be split into two parts:

1. The public part which serves our traffic to the general Internet public;
1. A second part for doing "administration", which will be updated separately from the public part;

This application will have a few points at which it is updated and pushed to GitHub. This will enable us to demonstrate CI and CD after we've made changes.

## Our Architecture: in brief
I have outlined the infrastructure below. Diagrams have also been provided, giving a visual representation.

Here is a brief breakdown of each component used to deliver our application.

### HTTPS Serving
* HAProxy load balancers: x2

### Application Servers
* Application x2
* Management Application x2

### Database
* PostgreSQL x2
* repmgr (replication manager) x2

### Monitoring:
* Sensu x2 
* Redis x2

### Logging
* ElasticSearch cluster (two nodes)
* Logstash: x2

Kibana will be used too, but it will be installed on each ElasticSearch instance. I am also weighing my options between [ElasticHQ](http://www.elastichq.org) and [Kopf](https://github.com/lmenezes/elasticsearch-kopf).

