# teamd-systemd
## Teaming interfaces on boot (replace bonding)
------------------------------------

teamd-systemd is a systemd service which install a service in format
teamd@po0 and starts a daemon named teamd using /etc/teamd.d/po0.conf
configuration file. This file can contain a simple configuration like
this:

```
/etc/teamd.d/po0.conf:
{
    "device": "po0",
    "runner": {
       "name": "lacp",
       "active": true,
       "fast_rate": true,
       "tx_hash": ["vlan", "ipv4", "ipv6"] 
    },
    "link_watch": {
        "name": "ethtool"
    }
}
```

On start will create a teaming interface named po0. After that this 
interface can be used in networking scripts or from cli to 
add/remove ports like this:

CLI:
```
systemctl enable teamd@po0
systemctl start teamd@po0
teamdctl po0 port add eth0 eth1
```

Debian/Ubuntu network scripts:

```
systemctl enable teamd@po0
systemctl start teamd@po0
```

Edit /etc/networking/interfaces and add the configuration for interface po0:

```
auto po0
iface po0 inet manual
    pre-up    /usr/bin/teamdctl $IFACE port add eth0
    pre-up    /usr/bin/teamdctl $IFACE port add eth1
    up        /sbin/ip link set $IFACE up
    post-down /sbin/ip link set $IFACE down
```

And then fireup the po0:

```
ifup po0
```

## NOTE! The members interfaces eth0 and eth1 MUST be in SHUTDOWN state prior to add in team interface! Otherwise will receive an error!

