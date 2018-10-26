openldap
===============

This roles installs the OpenLDAP server or client the target machine. It has the
option to enable/disable SSL by setting it in defaults or overriding it.


Requirements
------------

This role requires Ansible 1.4 or higher, and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows:


```
#### LDAP AUTOMOUNT CONFIG
admin_ou: "ou=admin,{{ openldap_server_dc }}"
groups_ou:     "ou=groups,{{ openldap_server_dc }}"
users_ou:     "ou=users,{{ openldap_server_dc }}"
automount_ou: "ou=automount,{{ admin_ou }}"
auto_master_ou: "ou=auto.master,{{ automount_ou }}"
auto_data_ou: "ou=auto.data,{{ automount_ou }}"
auto_home_ou: "ou=auto.home,{{ automount_ou }}"


openldap_server_domain_name: "ldap.home.prettybaked.com"
openldap_server_ip: "192.168.1.15"

openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
openldap_server_enable_ssl: false

# The self signed ssl parameters
openldap_server_country: "US"
openldap_server_state: "colorado"
openldap_server_location: "denver"
openldap_server_organization: "Home"
```


Example Playbooks
--------

##### Configure an OpenLDAP server:
```
- hosts: ldapservers
  become: True
  tasks:
    - import_role:
        name: common
    - import_role:
        name: openldap
  vars:
    - openldap_server_domain_name: "ldap.home.prettybaked.com"
    - openldap_server_ip: "192.168.1.15"
    - openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
```

##### Deploy OpenLDAP Server with users, ssh keys, autoFS configurations with NFS or CephFS storage backend:
```
- hosts: ldapservers
  become: True
  tasks:
    - import_role:
        name: common
    - import_role:
        name: openldap
    - import_role:
        name: ceph-fs
      when: storage_backend == "cephfs"
    - import_role:
        name: nfs
      when: storage_backend == "nfs"
  vars:
    - openldap_server_domain_name: "ldap.home.prettybaked.com"
    - openldap_server_ip: "192.168.1.15"
    - openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
```

##### Deploy LDAP Client configurations:
```
- hosts: all
  become: True
  tasks:
    - import_role:
        name: common
    - import_role:
        name: openldap
  vars:
    - openldap_server_domain_name: "ldap.home.prettybaked.com"
    - openldap_server_ip: "192.168.1.15"
    - openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
```

##### Deploy LDAP Client configurations with NFS or CephFS storage backend:
```
- hosts: all
  become: True
  tasks:
    - import_role:
        name: common
    - import_role:
        name: openldap
    - import_role:
        name: ceph-fs
      when: storage_backend == "cephfs"
    - import_role:
        name: nfs
      when: storage_backend == "nfs"
  vars:
    - openldap_server_domain_name: "ldap.home.prettybaked.com"
    - openldap_server_ip: "192.168.1.15"
    - openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
```


Dependencies
------------

None



