# InfluxDB
We need a time series database for holding our metrics. InfluxDB is my choice. It's a new technology and needs time to mature a bit more, but it currently offers a very plug-and-play set of tools which I've found to be more than stable and sufficient at this point in time.

## Architecture
InfluxDB works in a classic architecture pattern: client/server. The server part is obviously InfluxDB's role, but the client can be one of many offerings:

* Telegraf (my choice);
* Sensu;
* Nagios;

And virtually anything that can push to InfluxDB using its HTTPS API or UDP, implementing its (simple) line protocol.

We will use Telegraf.

### Redundancy & High Availability
The official documentation already covers the architecture we will cover. See [this document](https://docs.influxdata.com/influxdb/v0.12/high_availability/relay/) regarding the highly available setup.

We will need two additional tools to make InfluxDB resilient to failure:

* Nginx
* Relay

Nginx is likely a tool you've heard of - it's a web server that can serve static content, web applications (Python, Ruby, and so on), and importantly for us, act as a reverse proxy for TCP, UDP and HTTP(S) connections.

Relay is another tool offered by InfluxData, but it's rather raw in it presentation at this point. We will address this.

In short, we will use Relay to duplicate `/write` requests to the InfluxDB instances, and Nginx to direct `/query` requests around Relay and directly into InfluxDB. This means we can add and remove InfluxDB instances as and when we want to make the solution more highly available. With a smart data rentention, backup, and recovery plan, we can survive failure in this design by simply copying the shards from a healthy server to a new or recovered server, all with zero downtime. 

## Installation
Installing InfluxDB is incredibly easy on CentOS. It simply involves downloading and installing an RPM, defining a simple configuration file, and starting the service. In my experience so far, it has been very reliable in this regard.

### Steps

1. Download the RPM from InfluxData;
1. Define the configuration you need, paying attention to:
  1. Your retention policy;
  1. TLS security;
  1. Authentication setup: create an admin user;
1. Configure your firewall to allow:
  1. HTTPS and/or;
  1. UDP;
1. Start the service;

You should have a running, working InfluxDB waiting to accept data.

### Storage
I will be using an additional HDD attached to each InfluxDB instance. This disk will persistent the deletion of a VM for safety reasons, and it can also be reattached to a new VM in the event the original fails or you wish to move the disk to another box for other reasons.

### Ansible Playbook
A role will be developed in time, but for the time being this Playbook will installa InfluxDB and upload a configuration.

```yaml
- name: Install InfluxDB
  hosts: influxdb 
  become: true
  tags:
    - influxdb
  tasks:
    - name: Install InfluxDB
      yum:
        name: "https://dl.influxdata.com/influxdb/releases/influxdb-0.12.2-1.x86_64.rpm"
        state: present

    # You will need to provide this file
    - name: Upload InfluxDB configuration
      copy:
        src: "influxdb.conf"
        dest: /etc/influxdb/influxdb.conf

    - name: Start services
      service:
        name: "{{item}}"
        state: started
        enabled: true
      with_items:
        - influxdb
        - chronograf
```
