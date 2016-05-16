    # Architecture
I want to briefly talk about architecture here. This will set an understanding as to what we're trying to achieve with the various documentation and code bases we will explore.

It's important to remember that the goal of this book is to eventually discuss various aspects of architecture and which kind of designs you can adopt to deploy and deliver your products and services. However that being said, at this point in time we will cover a single architecture for a monolithic web application.

## The Monolithic Application
A monolithic application is a large, all-in-one code base made up of tightly coupled components that cannot work without each other. In such software projects it can often be difficult to separate components or replace them, but this doesn't always hold true. A well designed and maintained monolithic code base can, and often will, perform very well and be quite maintainable well into its life cycle.

The monolithic software design is also extremely common at the time of writing. In fact many experienced and well respected software engineers will suggest starting out with a large, single application before breaking it up into a micro service architecture.

## The Real World
In order to demonstrate the knowledge and tools being discussed throughout this book, we will look at deploying a real world application that many people will be familiar with and might even have an interest in hosting themselves: Wordpress.

Wordpress is an incredibly popular blogging platform, used the world over. A lot of people, mainly systems administrators who have to maintain the installation, would say Wordpress is a pain to keep up to date and is constantly being attacked/exploited. I believe this makes it the perfect candidate for this book's goals.

## The Architecture
What will the architecture look like? It will be relatively simple by design, but complete in its implementation. It will have monitoring, backups, redundant databases, load balancers, static content caching (not via CDNs, but using Varnish) and so forth.

Let's break down the components into a list and include some numbers.

### HTTPS Serving
* HAProxy load balancers: x3
* Varnish static content caches: x3
* NginX application/static content servers: x3

### PHP
* PHP-FPM servers (PHP-FPM over TCP): x3

### Database
* MariaDB with Galera: x3

### Monitoring
This topic is current up for debate. I am considering three solutions, all of which have their merits:

* Sensu + Redis;
* Telegraf + InfluxDB;
* Prometheus;

#### Sensu + Redis
This solution is very good. It as a mature community and has been around a while now. Its architecture is also very smart and scalable, but a few things personally bug me:

* The pre-defined checks, as well as most community checks, are written in Ruby. I have no desire to manage Ruby code;
* It requires several Sensu Mastes and a Redis Cluster, doubling up on the infrastructure needed versus other options;
* Sensu and Redis are two individual projects and this is probably fine, but a major change in one could break the other;

#### Telegraf + InfluxDB
This stack is a much more plug & play solution and each component comes form the one ecosystem: InfluxData. These guys have created the [TICK stack](https://influxdata.com/get-started/what-is-the-tick-stack/).

I've had good experiences with the TICK stack, but sadly one hughely critical aspect of InfluxDB practically takes it off the table: [clustering/replication for resilience is a $400/month proprietary add-on]().

I don't know about you, dear reader, but data integrity and security are two aspects of software design which should by default, for free.

#### Prometheus
This is a other all-in-one, plug & play solution written in Go. This is a very strong contender due to the single ecosystem solution it presents.

### Logging
* ElasticSearch cluster: x3
* Logstash: x3

Kibana will be used too, but it will be installed on each ElasticSearch instance.

I am also weighing my options between [ElasticHQ](http://www.elastichq.org) and [Kopf](https://github.com/lmenezes/elasticsearch-kopf).
