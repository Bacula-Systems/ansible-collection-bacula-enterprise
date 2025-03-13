# Bacula Enterprise Collection for Ansible - baculasystems.bacula_enterprise

# bweb role

## [Example Playbook](#example-playbook)

```yaml
---
- name: Converge
  hosts: bweb_host
  become: true
  gather_facts: true

  roles:
    - role: bweb
      bweb_server_bind: all
```

## [Role Variables](#role-variables)

```yaml
bweb_server_bind: localhost
bweb_server_port: 9180

# SSL - function default: false.
# If true, the bweb_ssl_pem_file variable must be set to an pem file,
# with private key and cert combined. The file must be set to mode 400 and
# the ownership must be set to
# bweb_server_username:bweb_server_groupname.
bweb_ssl_enabled: false
bweb_ssl_pem_file: /opt/bweb/etc/server.pem

bweb_ssl_key_file: /etc/ssl/private/key.pem
bweb_ssl_cert_file: /etc/ssl/certs/cert.pem

# User and Group
bweb_server_username: bacula
bweb_server_groupname: bacula

# Authentification
# could be set to http_basic
bweb_auth_type: none
bweb_auth_file: /opt/bweb/etc/htpasswd.bweb
```

## [Requirements](#requirements)

Installed bacula Bweb.

## [Context](#context)

## [Compatibility](#compatibility)

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

## [Changelog](#changelog)

## [License](#license)

## [Author Information](#author-information)
