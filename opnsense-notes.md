# my opnsense notes
Since I want to behave opnsense like linux, i created this hack to avoid complex wazuh configs
This is just a PoC, thats an ugly hack, do not use that in prod.

```
root@light:/usr/local/opnsense/service/conf/actions.d # service configd restart
Stopping configd...done
Starting configd.
root@light:/usr/local/opnsense/service/conf/actions.d # cat /usr/local/opnsense/service/conf/actions.d
cat: /usr/local/opnsense/service/conf/actions.d: Is a directory
root@light:/usr/local/opnsense/service/conf/actions.d # cat /usr/local/opnsense/service/conf/actions.d/actions_wazuh_agent_custom.conf
[start]
command:
    /bin/cp -af /var/ossec/active-response/bin/opnsense-fw /var/ossec/active-response/bin/firewall-drop
type:script
message:wazuh link firewall-drop to opnsense-fw
root@light:/usr/local/opnsense/service/conf/actions.d # service configd restart
Stopping configd...done
Starting configd.

# now reconfigure cron job
```
