# Monitoring
Obviously this is an important subject. All networks should have well defined monitoring in place to protect against long-term or unnoticed failure of resources. Monitoring can even go as far as allowing you to implement automatic (de)scaling of your network, something that may be covered in the future.

## Coverage
At minimum, I believe a monitoring solution should cover:

* Uptime;
* Logged in users;
* CPU, RAM, disk, and networking usage;
* Process monitoring;

These items alone allow for a very broad spectrum of problems to be diagnosed and resolved, so it's what I want to implement, at minimum, in all networks.

## The Tools
The InfluxData is a new ecosystem of tools for capturing and working with time series data. It's relatively new, but in its current state it's excellent for collecting all kinds of metrics from all kinds of software stacks; it's very easy to setup, make highly available, and begin graphing the data in not a lot of time, as I will demonstrate.

I won't go into the details on InfluxData or try to explain it's function or form here. The developers have done a good job of that themselves. More information can be [found here](https://influxdata.com/time-series-platform/influxdb/).

InfluxData is made up of:

* InfluxDB
* Telegraf
* Chronograf

I will discuss each tool one at a time.

