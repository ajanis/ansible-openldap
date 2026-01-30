# LDAP server-side commands for SSSD (sudo + autofs)

This role now treats LDAP as *already managed elsewhere* and only configures clients (SSSD, NSS, PAM, sshd, autofs).

Use these commands from a machine with `ldap-utils` (Ubuntu) or `openldap-clients` (RHEL). Adjust `-H`, `-D`, and the search bases to match your environment.

## Verify sudo rules are visible

```bash
ldapsearch -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W \
  -b "ou=sudoers,dc=home,dc=prettybaked,dc=com" '(objectClass=sudoRole)' \
  cn sudoUser sudoHost sudoCommand sudoOption
```

## Create sudo OU (if missing)

```bash
cat > sudoers-ou.ldif <<'LDIF'
dn: ou=sudoers,dc=home,dc=prettybaked,dc=com
objectClass: top
objectClass: organizationalUnit
ou: sudoers
LDIF

ldapadd -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W -f sudoers-ou.ldif
```

## Example sudoRole entries

### Passwordless sudo for %admin

```bash
cat > sudo-admin.ldif <<'LDIF'
dn: cn=admin,ou=sudoers,dc=home,dc=prettybaked,dc=com
objectClass: top
objectClass: sudoRole
cn: admin
sudoUser: %admin
sudoHost: ALL
sudoCommand: ALL
sudoOption: !authenticate
LDIF

ldapadd -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W -f sudo-admin.ldif
```

### Allow user jdoe to restart a service

```bash
cat > sudo-jdoe-systemctl.ldif <<'LDIF'
dn: cn=jdoe-systemctl,ou=sudoers,dc=home,dc=prettybaked,dc=com
objectClass: top
objectClass: sudoRole
cn: jdoe-systemctl
sudoUser: jdoe
sudoHost: ALL
sudoCommand: /bin/systemctl restart docker
LDIF

ldapadd -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W -f sudo-jdoe-systemctl.ldif
```

## Verify automount maps are visible

```bash
ldapsearch -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W \
  -b "ou=auto.master,dc=home,dc=prettybaked,dc=com" '(objectClass=automountMap)' ou

ldapsearch -x -H "ldap://openldap.home.prettybaked.com" -D "cn=admin,dc=home,dc=prettybaked,dc=com" -W \
  -b "ou=auto.master,dc=home,dc=prettybaked,dc=com" '(objectClass=automount)' cn automountInformation
```

## Recommended client-side SSSD settings (for reference)

- Sudo: ensure SSSD has `services = ... sudo ...` and the domain has `sudo_provider = ldap` and `sudo_search_base = ...`.
- Autofs: ensure SSSD has `services = ... autofs ...` and the domain has `autofs_provider = ldap` and `ldap_autofs_search_base = ...`.

## Security note

Avoid using an LDAP admin bind DN/password in `sssd.conf` on clients. Prefer a dedicated read-only bind account with access limited to the needed bases (users/groups, sudoers, automount, ssh keys).
