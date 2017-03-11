## ERL-Dashboard
###Grafana Dashboard for EdgeRouter Lite
Using Grafana, InfluxDB and collectd to graph metrics.
Dashboard preview on [raintank.io](https://snapshot.raintank.io/dashboard/snapshot/NytKSRUemSB4ZML10wcjqXmgt5J1aZ64)
... or on [imgur](http://imgur.com/sxVC2lp)

Pretty much all of the credit for this goes to other users from /r/homelab. 

#### collectd

I am running grafana, influxdb, and collectd on one Debian VM. 

For this to work, collectd should be at version 5.6 or better. Debian ships with 5.4 so you will have to install from the collectd repo or source. On centos, make sure you install the collectd-snmp package.

 
 



