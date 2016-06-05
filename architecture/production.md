# Production
When we deliver to the public Internet, we will use a production class infrastructure. This is an environment which must be able to withstand the inbound traffic, stand up even after a system fails, and keep track of what's going on within the application and network.

Here is a basic visual representation of production:

![Production](http://i.imgur.com/yASDD5G.png)

The diagram contains numbered sections. I will discuss these below.

## 1: Inbound, public traffic
- All traffic will come in over HTTPS, terminated at the HAProxy instances;
- Only port 443 will be open;
- A [Let's Encrypt]() certificate will be used;

## 1.1: Secondary DNS A records
- HAProxy instances won't be clustered or know of each other's existence (no `keepalived`);
- Modern browsers re-try connections using all DNS results that come back from a domain lookup;
- The client's browser will take care of a failed HAProxy box, as stated above;

## 2: Internal, application traffic
- Internal traffic will also be done over HTTPS;

## 3: Database communications
- TLS connections will talk to replication manager directly;
- A failover replication manager box is available and should be utilised by applications;

## 4: Database clustering
- repmgr version 3 is used for advanced PostgreSQL replication and clustering;
- repmgr is very smart and can be configured in a variety of ways;

## 4.1: Database replication
- Built in replication is used to keep data safe;
- Master/Slave setup in this case, not Master/Master;

## 5: Sensu Agent
- The Sensu agent on each system talks to Redis' pub/sub feature and publishes check results;
- We will have system level checks and all the common things you would expect;
- We will also have checks that cover our application and other services needed for operation;

## 6: Sensu Master
- The Sensu master system is (primarily) responsible for taking checks from Redis and reporting on them;
- Multiple Sensu masters are used for resilience;

## 7: Application logging
- All systems push everything `rsyslog` receives to LogStash;
- Our application will have different log levels to allow depth for debugging;

## 8: Log storage, indexing, and visualisations
- LogStash mutates the input and pushes it to ElasticSearch;
- ElasticSearch then indexes the logs and Kibana visualises them;

## 9: Bastion hosts
- Bastion hosts allow us to keep the SSH daemon/port for each server off of the Internet;
- We can proxy via bastion hosts to internal-only systems, such as management applications;