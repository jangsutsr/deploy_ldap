# deploy_ldap

This playbook gives you a quick way of setting an ldap backend up and running using awx container, alone with needed Tower configurations.

1. Have a Ubuntu 14 machine ready, alone with the credential needed to ssh onto it. (For example, spawn an ec2 Ubuntu 14 instance) Make sure TCP ports 22, 636 and 389 are open to the world.

2. In AWX, create a git project using this repo: https://github.com/jangsutsr/deploy_ldap.git.

3. In AWX, create an inventory containing the Ubuntu 14 machine in step 1, and a machine credential against the credential in step 1.

4. In AWX, create a job template using the project, inventory and credential created. Note 'Enable Privilege Escalation' should be checked also.

5. Launch the job template to deploy LDAP backend on the Ubuntu 14 machine.

6. Configure your AWX to use the LDAP backend by PATCH to `/api/v2/settings/ldap/`:
```
{
    "AUTH_LDAP_SERVER_URI": "ldap://<name of you Ubuntu 14 machine>",
    "AUTH_LDAP_BIND_DN": "cn=Manager,dc=example,dc=com",
    "AUTH_LDAP_BIND_PASSWORD": "fo0m4nchU",
    "AUTH_LDAP_START_TLS": false,
    "AUTH_LDAP_CONNECTION_OPTIONS": {
        "OPT_NETWORK_TIMEOUT": 30,
        "OPT_REFERRALS": 0
    },
    "AUTH_LDAP_USER_SEARCH": [
        "ou=people,dc=example,dc=com",
        "SCOPE_SUBTREE",
        "(cn=%(user)s)"
    ],
    "AUTH_LDAP_USER_DN_TEMPLATE": null,
    "AUTH_LDAP_USER_ATTR_MAP": {
        "first_name": "givenName",
        "last_name": "sn",
        "email": "mail"
    },
    "AUTH_LDAP_GROUP_SEARCH": [
        "dc=example,dc=com",
        "SCOPE_SUBTREE",
        "(objectClass=groupOfNames)"
    ],
    "AUTH_LDAP_GROUP_TYPE": "GroupOfNamesType",
    "AUTH_LDAP_REQUIRE_GROUP": null,
    "AUTH_LDAP_DENY_GROUP": null,
    "AUTH_LDAP_USER_FLAGS_BY_GROUP": {
        "is_superuser": "cn=superusers,ou=groups,dc=example,dc=com"
    },
    "AUTH_LDAP_ORGANIZATION_MAP": {
        "LDAP Organization": {
            "admins": "cn=engineering_admins,ou=groups,dc=example,dc=com",
            "remove_admins": false,
            "users": [
                "cn=engineering,ou=groups,dc=example,dc=com",
                "cn=sales,ou=groups,dc=example,dc=com",
                "cn=it,ou=groups,dc=example,dc=com"
            ],
            "remove_users": false
        }
    },
    "AUTH_LDAP_TEAM_MAP": {
        "LDAP Sales": {
            "organization": "LDAP Organization",
            "users": "cn=sales,ou=groups,dc=example,dc=com",
            "remove": true
        },
        "LDAP IT": {
            "organization": "LDAP Organization",
            "users": "cn=it,ou=groups,dc=example,dc=com",
            "remove": true
        },
        "LDAP Engineering": {
            "organization": "LDAP Organization",
            "users": "cn=engineering,ou=groups,dc=example,dc=com",
            "remove": true
        }
    }
}
```

7. Test deploy by logging in LDAP users. The LDAP organization layout is as follows:
```
LDAP Organization
  username: super_user1
  password: password

  LDAP Engineering
    username: eng_user1
    password: password
    
    username: eng_user2
    password: password
    
    username: eng_admin1
    password: password

  LDAP IT
    username: it_user1
    password: password
    
    username: it_user2
    password: password
  
  LDAP Sales
    username: sales_user1
    password: password
    
    username: sales_user2
    password: password
```
