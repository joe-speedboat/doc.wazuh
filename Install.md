# Install Wazuh
* Install Rocky 9 minimal
  * Keep SELinux enabled
  * Keep FirewallD enabled

## Base OS Config
```
for i in 443/tcp 1514/tcp 1515/tcp 514/tcp 514/udp
do
  firewall-cmd --add-port=$i
  firewall-cmd --add-port=$i --permanent
done

dnf -y upgrade
reboot
```

## Install Wazuh single node
```
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

echo '#!/bin/bash
# remove old wazuh logs
KEEP_ARCHIVE=180
KEEP_ALERT=180
find /var/ossec/logs/alerts/ -type f -mtime +$KEEP_ALERT -exec rm -f {} \;
find /var/ossec/logs/archives/ -type f -mtime +$KEEP_ARCHIVE -exec rm -f {} \;
' > /etc/cron.daily/99_wazuh_cleanup
chmod 700 /etc/cron.daily/99_wazuh_cleanup
```

### Dashboard Session Timeout
```
vi /etc/wazuh-dashboard/opensearch_dashboards.yml
------
# The ttl is calculated in milliseconds
# 86400000 ms is 24 hrs
opensearch_security.cookie.ttl: 86400000
opensearch_security.session.ttl: 86400000
opensearch_security.session.keepalive: true
------
systemctl restart wazuh-dashboards
```

### Basis App Configuration
```
echo 'export PATH=$PATH:/var/ossec/bin' >> $HOME/.bashrc
. $HOME/.bashrc

agent_groups -a -g os_bsd -q
agent_groups -a -g os_debian -q
agent_groups -a -g os_redhat -q
agent_groups -a -g os_alpine -q
agent_groups -a -g os_windows -q
agent_groups -a -g app_kvm -q
#agent_groups -a -g app_docker -q
agent_groups -a -g app_jellyfin -q
agent_groups -a -g net_wan -q
agent_groups -a -g os_bsd -q
agent_groups -a -g os_bsd -q
agent_groups -a -g os_bsd -q
agent_groups -a -g os_bsd -q

cd /var/ossec/etc/shared/
echo '<agent_config>
  <rootcheck>
    <ignore>/var/lib/rancher/k3s</ignore>
    <ignore>/var/lib/kubelet/pods</ignore>
    <ignore>/var/lib/docker/overlay2</ignore>
  </rootcheck>
</agent_config>' > app_docker/agent.conf

echo '<agent_config>
    <localfile>
      <location>/var/log/jellyfin/jellyfin.log</location>
      <log_format>syslog</log_format>
    </localfile>
</agent_config>' > app_jellyfin/agent.conf

echo '<agent_config>
  <localfile>
    <location>/var/log/messages</location>
    <log_format>syslog</log_format>
  </localfile>
</agent_config>' > os_alpine/agent.conf

echo '<agent_config>
  <localfile>
    <location>journald</location>
    <log_format>journald</log_format>
  </localfile>
  <localfile>
    <location>/var/log/auth.log</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/syslog</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/dpkg.log</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/kern.log</location>
    <log_format>syslog</log_format>
  </localfile>
</agent_config>' > os_debian/agent.conf

echo '<agent_config>
  <localfile>
    <location>journald</location>
    <log_format>journald</log_format>
  </localfile>
  <localfile>
    <location>/var/log/messages</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/secure</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/maillog</location>
    <log_format>syslog</log_format>
  </localfile>
  <localfile>
    <location>/var/log/audit/audit.log</location>
    <log_format>audit</log_format>
  </localfile>
</agent_config>' > os_redhat/agent.conf




