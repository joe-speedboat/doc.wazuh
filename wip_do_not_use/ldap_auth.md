# Configure LDAP auth (FreeIPA example)
* https://documentation.wazuh.com/current/user-manual/user-administration/ldap.html
```
DS=directory01.sun.domain.tld
echo -n | openssl s_client -connect $DS:636 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/wazuh-indexer/opensearch-security/ldapcacert.pem

cat /etc/wazuh-indexer/opensearch-security/ldapcacert.pem

chown wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/opensearch-security/ldapcacert.pem

vim /etc/wazuh-indexer/opensearch-security/config.yml
------
authc:
  ldap:
    description: "Authenticate via LDAP or Active Directory"
    http_enabled: true
    transport_enabled: false
    order: 5
    http_authenticator:
      type: basic
      challenge: false
    authentication_backend:
      type: ldap
      config:
        enable_ssl: true #Set to true if LDAPS is enabled, otherwise set to false.
        pemtrustedcas_filepath: /etc/wazuh-indexer/opensearch-security/ldapcacert.pem #Required when enable_ssl is set to true
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: true
        hosts:
        - directory01.domain.tld:636 #Port 389 for LDAP, 636 for LDAPS
        - directory02.domain.tld:636 #Port 389 for LDAP, 636 for LDAPS
        bind_dn: uid=wazuh-bind,cn=users,cn=accounts,dc=domain,dc=tld
        password: 'xxx'
        userbase: 'cn=users,cn=accounts,dc=domain,dc=tld'
        usersearch: (cn={0})  #Depending on your LDAP schema this can be CN, sAMAccountName, etc
        username_attribute: cn

authz:
  roles_from_myldap:
    description: "Authorize via LDAP or Active Directory"
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
        enable_ssl: true #Set to true if LDAPS is enabled, otherwise set to false.
        pemtrustedcas_filepath: /etc/wazuh-indexer/opensearch-security/ldapcacert.pem #Required when enable_ssl is set to true
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: true
        hosts:
        - directory01.domain.tld:636 #Port 389 for LDAP, 636 for LDAPS
        - directory02.domain.tld:636 #Port 389 for LDAP, 636 for LDAPS
        bind_dn: uid=wazuh-bind,cn=users,cn=accounts,dc=domain,dc=tld
        password: 'xxx'
        userbase: 'cn=users,cn=accounts,dc=domain,dc=tld'
        usersearch: (cn={0}) #Depending on your LDAP schema this can be cn, sAMAccountName, etc
        username_attribute: cn
        rolebase:  cn=groups,cn=accounts,dc=domain,dc=tld #This is the subtree in the directory that contains the role/group
        rolesearch: '(member={0})' #Depending on your LDAP schema this can be member, memberOf, etc
        userrolename: memberof
        rolename: cn
        skip_users:
          - admin
------

export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /etc/wazuh-indexer/opensearch-security/config.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h 127.0.0.1 -nhnv


vim /etc/wazuh-indexer/opensearch-security/roles_mapping.yml
------
all_access:
  reserved: false
  hidden: false
  backend_roles:
  - "admin"
  - "wazuhadmin"
  description: "Maps admin to all_access"
------

export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /etc/wazuh-indexer/opensearch-security/roles_mapping.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h 127.0.0.1 -nhnv

cat /usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml
# run_as: true

systemctl restart wazuh-dashboard

```
