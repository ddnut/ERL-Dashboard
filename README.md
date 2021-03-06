##Grafana Dashboard for EdgeRouter Lite
Using Grafana, InfluxDB, SNMP and collectd to graph metrics from EdgeRouters and VyOS.

Preview available at [raintank.io](https://snapshot.raintank.io/dashboard/snapshot/NytKSRUemSB4ZML10wcjqXmgt5J1aZ64) or [imgur](http://imgur.com/sxVC2lp)

Pretty much all of the credit for this dashboard goes to other users from /r/homelab. The configuration and dashboard will require some tweaking to work in your environment.
In my case, I monitoring both my ERL and a VyOS router I have running at another location. In addition to the 3 interfaces on my edgerouter lite, I am also monitoring a site-to-site openvpn connection (vtun1.)
You may get errors from collectd if you do not remove the additional interfaces from [conf/10-snmp.conf](conf/10-snmp.conf)

###EdgeRouter configuration
see also: [conf/erl-snmp.conf](conf/erl-nsmp.conf)

SNMP Configuration on an EdgeRouter Lite or vyos is pretty straight forward. 

From configuration mode, enter the following commands, modify as required
```
set service snmp community 'mycommunity'
set service snmp contact 'admin@domain.org'
set service snmp listen-address 'ip'
set service snmp v3 group mygroup mode 'ro'
set service snmp v3 group mygroup seclevel 'auth'
set service snmp v3 group mygroup view 'myview'
set service snmp v3 user myuser auth plaintext-key 'password'
set service snmp v3 user myuser auth type 'md5'
set service snmp v3 user myuser group 'mygroup'
set service snmp v3 user myuser mode 'ro'
set service snmp v3 view myuser oid '1'
commit
save
```

Your configuration should end up looking something like this:
```
service {
    snmp {
        community mycommunity {
        }
        contact admin@company.org
        listen-address your-ip {
        }
        v3 {
            group mygroup {
                mode ro
                seclevel auth
                view myview
            }
            user myuser {
                auth {
                    plaintext-key password
                    type md5
                }
                group mygroup
                mode ro
            }
            view myview {
                oid 1 {
                }
            }
        }
    }
}
```

###collectd

I am running grafana, influxdb, and collectd on one Debian VM. 

For this setup to work, collectd should be at version 5.6 or better. Debian ships with 5.4 so you will have to install from the collectd repo or source. On CentOS, make sure you install the collectd-snmp package.

####conf/net-influxdb.conf
[conf/net-influxdb.conf](conf/net-influxdb.conf)

This collectd configuration file specifies the InfluxDB host that you'll be shipping metrics to. 
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
[conf/10-snmp.conf](conf/10-snmp.conf)

This file contains collectd SNMP plugin configuration and maps SNMP OID/MIB data to something InfluxDB can use. A redditor from /r/homelab originally posted this; I would find the post and give credit, but reddit's search is broken again (surprise!)

I have noted in the file where you should make changes. If you have other interfaces or things you want to monitor, you can probably use snmpwalk to find the snmp oid.

For example, this is how I found the vtun1 interface on my VyOS router:

```
snmpwalk -v3 -l  authNoPriv -u username -a MD5 -A Password RouterIP > rtr.txt

cat rtr.txt | grep vtun1
iso.3.6.1.2.1.2.2.1.2.10 = STRING: "vtun1"
iso.3.6.1.2.1.25.3.2.1.3.262154 = STRING: "network interface vtun1"
iso.3.6.1.2.1.25.4.2.1.5.17092 = STRING: "--daemon openvpn-vtun1 --verb 3 --writepid /var/run/openvpn-vtun1.pid --status /opt/vyatta/etc/openvpn/status/vtun1.status 30 --"
iso.3.6.1.2.1.31.1.1.1.1.10 = STRING: "vtun1"
iso.3.6.1.2.1.31.1.1.1.18.10 = STRING: "vtun1"
```

###grafana

To begin using this dashboard, import dashboard.json as a new dashboard. You will need to modify the datasource and host for each panel. 
This can be done through the Grafana UI or by modifying dashboard.json with the appropriate values. 

Note: The units and some of the math may not be configured correctly. I have not had the chance to go back and play with this.




