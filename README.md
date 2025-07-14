# openldap

<!-- MarkdownTOC -->

- Dependencies
- Requirements
- Role Variables
  - defaults/main.yml
  - vars/Debian.yml
  - vars/RedHat.yml
- Example Group Variables
- Example Playbooks
    - Configure an OpenLDAP server:
- License
- Author Information

<!-- /MarkdownTOC -->

This roles installs the OpenLDAP server or client the target machine.

The following installation / configuration options are available:

- Schema for storing SSH keys in ldap
- Schema for using LDAP as a backend to Samba
- Schema for storing AutoFS mount options in LDAP
- Optionally, configure a Samba server that uses the LDAP server for AuthN/AuthZ
- Optionally, configure SSH users and groups
- Optionally, configure AutoFS mounts for home directories and shared storage (NFS or CephFS)

## Dependencies
None

## Requirements

This role requires Ansible 2.8 or higher, and platform requirements are listed
in the metadata file.

Inventory Groups required for deploying Server components:
- ldapserver: OpenLDAP Server
- samba: Samba Server
- afpd: Apple File Protocol Daemon (Netatalk) + Avahi

## Role Variables

The variables that can be passed to this role and a brief description about
them are as follows:

### defaults/main.yml
```
openldap_server_domain_name: "ldap.home.example.com"
openldap_server_ip:

# shared storage flag
shared_storage: False


# These normally do not need to be changed
openldap_server_ldif: domain.ldif
openldap_server_dc: "dc={{ openldap_server_domain_name.split('.')[0] }},dc={{ openldap_server_domain_name.split('.')[1] }},dc={{ openldap_server_domain_name.split('.')[2] }},dc={{ openldap_server_domain_name.split('.')[3] }}"

# Bind DN
openldap_server_bind_dn: "cn=Manager,{{ openldap_server_dc }}"


# AutoFS configs
openldap_group_ou:     "ou=groups,{{ openldap_server_dc }}"
openldap_user_ou:     "ou=users,{{ openldap_server_dc }}"
openldap_autofs_search_base: "{{ openldap_search_base }}"
openldap_sudo_search_base: "{{ openldap_search_base }}"
openldap_auto_master_ou: "ou=auto.master,{{ automount_ou }}"
openldap_auto_data_ou: "ou=auto.data,{{ automount_ou }}"
openldap_auto_home_ou: "ou=auto.home,{{ automount_ou }}"

data_mount_root: "/data"
ldap_user_home_directory: "homedirs"



openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
openldap_server_enable_ssl: false

ssl_certpath:
ssl_keypath:
ssl_privkey:
ssl_certchain:

## The self signed ssl parameters
openldap_server_country: "US"
openldap_server_state: "colorado"
openldap_server_location: "denver"
openldap_server_organization: "Home"

ssh_users:
  ajanis:
    password: "{{vault_ajanis_pw}}"
    cn: "Alan Janis"
    givenname: "Alan"
    sn: "Janis"
    mail: "alan.janis@example.com"
    gecos: ajanis
    uid: 1043
    gid: 1042
    pubkey: >
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC29plb9A6fagU+g+zFX6Bo6SUGj58Z/6nxngV/
      ...
      ...
      XTrZ4lsZQjr8dGSI4HMqw== ajanis
    state: present

ssh_groups:
  admin:
    description: Administrators Group
    gid: 1042
    members:
      - ajanis

nfs_mounts:


smb_shares:
  - name: logs
    comment: "deploy logs"
    path: "/var/log/deploy"
    writeable: yes
    browseable: yes
    read_only: no
    valid_users: 'admin'

```

### vars/Debian.yml
```
openldap_server_pkgs:
  - slapd
  - ldap-utils
  - python-selinux
  - openssl
  - python-pip
  - libsasl2-dev
  - python-dev
  - libldap2-dev
  - libssl-dev
  - python-ldap
  - python-ldap3
  - python-pexpect

openldap_client_pkgs:
  - libnss-ldapd
  - libsasl2-dev
  - python-dev
  - libldap2-dev
  - libssl-dev
  - libpam-ldap
  - nscd
  - ldap-utils
  - python-pip
  - python-ldap
  - python-ldap3

samba_pkgs:
  - smbldap-tools
  - samba

openldap_server_app_path: "/etc/ldap"
openldap_server_user: "openldap"
openldap_nslcd_group: "nslcd"
```

### vars/RedHat.yml
```
openldap_server_pkgs:
  - openldap-servers
  - openldap-clients
  - compat-openldap
  - libselinux-python
  - openssl
  - openssl-devel
  - python-pip
  - python-ldap
  - python-ldap3
  - python-pexpect


openldap_client_pkgs:
  - nss-pam-ldapd
  - nscd
  - compat-openldap
  - python-devel
  - gcc
  - openldap-devel
  - cyrus-sasl
  - cyrus-sasl-ldap
  - cyrus-sasl-devel
  - ldap-utils
  - python-pip
  - python-ldap
  - python-ldap3

samba_pkgs:
  - smbldap-tools
  - samba

openldap_server_app_path: "/etc/openldap"
openldap_server_user: ldap

openldap_nslcd_group: ldap
```

## Example Group Variables

```
shared_storage: False

openldap_server_domain_name: "ldap.home.example.com"
openldap_server_ip: "10.0.10.15"
openldap_server_rootpw: "{{ vault_openldap_server_rootpw }}"
openldap_server_enable_ssl: false

ssl_privkey: "{{ vault_example_com_ssl_private_key }}"
ssl_keypath: "/etc/ssl/private/example.com.key"
ssl_certchain: "{{ vault_example_com_ssl_certificate }}"
ssl_certpath: "/etc/ssl/certs/example.com.crt"

ssh_users:
  ajanis:
    password: "{{vault_ajanis_pw}}"
    cn: "Alan Janis"
    givenname: "Alan"
    sn: "Janis"
    mail: "alan.janis@example.com"
    gecos: ajanis
    uid: 1043
    gid: 1042
    pubkey: >
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC29plb9A6fagU+g+zFX6Bo6SUGj58Z/6nxngV/
      ...
      ...
      XTrZ4lsZQjr8dGSI4HMqw== ajanis
    state: present

ssh_groups:
  admin:
    description: Administrators Group
    gid: 1042
    members:
      - ajanis

```

## Example Playbooks

##### Configure an OpenLDAP server:

```
- name: Deploy OpenLDAP Server with users, ssh keys, autoFS configurations
  hosts: ldapservers
  become: True
  tasks:
    - include_role:
        name: common
    - include_role:
        name: openldap
      when:  openldap_server_ip is defined and openldap_server_ip != None
    - include_role:
        name: ceph-fs
      when:
        - shared_storage
        - storage_backend == "cephfs"
    - include_role:
        name: nfs
      when:
        - shared_storage
        - storage_backend == "nfs"

```


## License

MIT

## Author Information

Created by Alan Janis
