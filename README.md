##Grafana Dashboard for EdgeRouter Lite
Using Grafana, InfluxDB, SNMP and collectd to graph metrics from EdgeRouters and VyOS.

Preview available at [raintank.io](https://snapshot.raintank.io/dashboard/snapshot/NytKSRUemSB4ZML10wcjqXmgt5J1aZ64) on [imgur](http://imgur.com/sxVC2lp)

Pretty much all of the credit for this dashboard goes to other users from /r/homelab. The configuration and dashboard will require some tweaking to work in your environment.
In my case, I monitoring both my ERL and a VyOS router I have running at another location. In addition to the 3 interfaces on my edgerouter lite, I am also monitoroing a site-to-site openvpn connection (vtun1.)
You may get errors from collectd if you do not remove the additional interfaces from [conf/net-influxdb.conf](conf/net-influxdb.conf)

###EdgeRouter configuration


###collectd

I am running grafana, influxdb, and collectd on one Debian VM. 

For this setup to work, collectd should be at version 5.6 or better. Debian ships with 5.4 so you will have to install from the collectd repo or source. On centos, make sure you install the collectd-snmp package.

####conf/net-influxdb.conf
This collectd configuration file specifies the InfluxDB hsot that you'll be shipping metrics to. 
```
#/etc/collectd/collectd.conf.d/net-influxdb.conf
<Plugin network>
  <Server "Hostname or IP" "Port"> 
		# Replace with your influxdb IP and Port
    SecurityLevel "none"
    Username "username" # influxdb user account with write privileges
    Password "password" # password for influxdb account
  </Server>
</Plugin>
```
 

####conf/10-snmp.conf
This file contains collectd SNMP plugin configuration and maps SNMP MIB data to something InfluxDB can use. A redditor from /r/homelab posted this sometime ago. I would find the post and give crdit, but reddit's search is broken again (suprise!)

I have noted in the file where you should make changes. If you have other interfaces or things you want to monitor, you can probably use snmpwalk to find the snmp mib.

For example, this is how i found the vtun1 interface on my VyOS router:

>snmpwalk -v3 -l  authNoPriv -u username -a MD5 -A Password RouterIP > rtr.txt

>cat rtr.txt | grep vtun1
>iso.3.6.1.2.1.2.2.1.2.10 = STRING: "vtun1"
>iso.3.6.1.2.1.25.3.2.1.3.262154 = STRING: "network interface vtun1"
>iso.3.6.1.2.1.25.4.2.1.5.17092 = STRING: "--daemon openvpn-vtun1 --verb 3 --writepid /var/run/openvpn-vtun1.pid --status /opt/vyatta/etc/openvpn/status/vtun1.status 30 --"
>iso.3.6.1.2.1.31.1.1.1.1.10 = STRING: "vtun1"
>iso.3.6.1.2.1.31.1.1.1.18.10 = STRING: "vtun1"



[conf/net-influxdb.conf](conf/net-influxdb.conf)

###grafana

To begin using this dashboard, import dashboard.json as a new dashboard. You will need to modify the datasource and host for each panel.
This can be done through the Grafana UI or by modifying dashboard.json with the appropriate values.

Note: The units and some of the math may not be configured correctly. I have not had the chance to go back and play with this.



