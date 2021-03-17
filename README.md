# Bacula Enterprise Collection for Ansible - bacula.bacula_enterprise

An Ansible Collection of roles to install Bacula Enterprise Edition - PostgreSQL Catalog, Bacula File Daemon only and BWeb in RHEL/CentOS/Debian/Ubuntu/OracleLinux/SLES.

## Ansible version compatibility

This collection has been tested against following Ansible versions: **>=2.9,<2.10**.

## Installing the Collection

Before using the Bacula Enterprise collection, you need to install it with the Ansible Galaxy CLI:

    ansible-galaxy collection install bacula.bacula_enterprise

## Roles

Role | Description
-------- | ---------------------
bee | install Bacula Enterprise Edition - PostgreSQL Catalog. This role may install PostgreSQL required packages if not present in the host.
bee_fdonly | install Bacula File Daemon only. This is a Bacula Client.
bee_sdonly | Install a Bacula Storage. This role follows the recommended procedure to install Bacula Enterprise Edition and stop/disable the Director.
bee_fdplugin | install a Bacula Enterprise Plugin in a Bacula Client host. Please note it is required to have a Bacula File Daemon installed and running before installing the File Daemon Plugin.
bee_sdplugin | install a Bacula Enterprise Plugin in a Bacula Storage host. Please note it is required to have a Bacula Storage Daemon installed and running before installing the Storage Daemon Plugin.
bee_plugindependencies | install Plugins required dependencies. This role is required and used by both the *bee_fdplugin* and the *bee_sdplugin* roles.

### Roles Variables

The roles have most of the variables defined in the `vars` directory or in the `default/main.yml` file.

Please set the repository URL for Bacula Enterprise Edition packages, `dl_area` variable, in the `default/main.yml` file for each role previously to use them.

You can use the following command to modify all `dl_area` values:

> find /root/.ansible/collections/ansible_collections/bacula/bacula_enterprise/roles/ -name main.yml -ls -exec sed -i 's/@@customer@/<your_customer_download_area_here>/g' {} \\;

The Bacula Client and the Bacula Storage will use the password used by the Director {} resource present in the bacula-fd.conf and bacula-sd.conf files, respectively, to communicate with the remote Director provided in the deployment.

The following variables must be set in the playbook, inventory file or by using `--extra-vars`, depending on the role used: `bee_version, director_hostname, client_hostname, client_name`, `storage_hostname`, `storage_name`, `volumes_directory` and `dedup_directories`.

Variable | Description
-------- | ---------------------
bee_version | the Bacula Enterprise Edition version, for example, `12.6.2`.
director_hostname | the FQDN of the Director host, for example, `baculadir.domain.com`. The name of the Director is, by default, the hostname + "-dir". In this example, the Director name will be `baculadir-dir`.
client_hostname | the FQDN of the Client host, for example, `client.domain.com`.
client_name | the name of the File Daemon/Client, for example, `client1-fd`. If not specified, the `client_name` will be the `client_hostname` + `-fd`.
storage_hostname | the FQDN of the Storage host, for example, `baculasd.domain.com`.
storage_name | the name of the Storage, for example, `storage1-sd`. If not specified, the `storage_name` will be the `storage_hostname + `-sd`.
volumes_directory | this variable will be required in both the `bee_sdonly` and `bee_sdplugin` roles. This is where the volumes will be stored, the value used in the Device {} `archive_device` directive.
dedup_directories | these are the Dedup Index and the Dedup Containers directories required when using the `bee_sdplugin` role to install the GED plugin. If not defined, the default values are `/opt/bacula/dedup/index` and `/opt/bacula/dedup/containers`. They can be modified using, for example, --extra-vars='{"dedup_directories": [/mnt/dedup/index, /mnt/dedup/containers]}'. Please note the first directory will be used for the Dedup Index and the second one, for the Dedup Containers.

### Usage and Examples

The following examples use the playbooks in the `tests` directory of the Bacula Enterprise collection.

1) To deploy Bacula Enterprise Edition (Director, Storage Daemon and File Daemon) `12.6.2` in the `baculadir.domain.com` host, using an inventory file:

> ansible-playbook -i tests/directors.yaml tests/bee.yml"

The `directors.yaml` file:

```
---
directors:
  hosts:
    baculadir.domain.com:
      director_hostname: baculadir.domain.com
      bee_version: 12.6.2
```

or using --extra-vars (please modify `- hosts: directors` to `- hosts: {{ director_hostname }}` in the `tests/bee.yml playbook):

> ansible-playbook tests/bee.yml --extra-vars "director_hostname=baculadir.domain.com bee_version=12.6.2"

2) To install two remote Bacula Enterprise Edition File Daemon only version `12.6.2` in the `client1.domain.com` and the `client2.domain.com` hosts and deploy the clients configuration in the `baculadir.domain.com-dir` Director in the `baculadir.domain.com` host:

> ansible-playbook -i tests\clients.yaml tests/bee_fdonly.yml

The `clients.yaml` file:

```
---
clients:
  hosts:
    client1.domain.com:
      client_hostname: client1.domain.com
    client2.domain.com:
      client_hostname: client2.domain.com
  vars:
    bee_version: 12.6.2
    director_hostname: baculadir.domain.com
```

3) To install a Bacula Enterprise Edition File Daemon Plugin `12.6.2` in the remote client in the `client.domain.com` host and deploy the plugin configuration (basic FileSet, not including plugin options in most of the cases) in the `baculadir.domain.com-dir` Director in the `baculadir.domain.com` host:

> ansible-playbook -i tests/storages.yaml tests/bee_fdplugin.yml --extra-vars "client_hostname=client.domain.com client_name=bacula-fd bee_version=12.6.2 fdplugin=mysql director_hostname=baculadir.domain.com"

4) To install two remote Bacula Enterprise Edition Storage Daemon only `12.6.2` in the `storage1.domain.com` and the `storage2.domain.com` hosts (Bacula Enterprise Edition will be installed and Director service will be manually stopped and disabled):

> ansible-playbook -i tests/storages.yaml tests/bee_sdonly.yml

The `storages.yaml` file:

```
---
storages:
  hosts:
    storage1.domain.com
      storage_hostname: storage1.domain.com
    storage2.domain.com
      storage_hostname: storage2.domain.com
  vars:
    director_hostname: baculadir.domain.com
    bee_version: 12.6.2
    volumes_directory: /opt/bacula/volumes
```

5) To install a Bacula Enterprise Edition Storage Daemon Plugin `12.6.2` in the `baculadir.domain.com` (the same host as Director in item 1):

> ansible-playbook -i inventory tests/bee_sdplugin.yml --extra-vars "storage_hostname=baculadir.domain.com storage_name=bacula-dedup-sd bee_version=12.6.2 sdplugin=dedup volumes_directory=/mnt/dedup/volumes director_hostname=baculadir.domain.com"

> ansible-playbook -i inventory tests/bee_sdplugin.yml --extra-vars '{"storage_hostname":"baculadir.domain.com",  "storage_name":"bacula-dedup-sd",  "bee_version":"12.6.2", "sdplugin":"dedup",  "volumes_directory":"/mnt/dedup/data/volumes",  "dedup_directories":"["/mnt/dedup/index", "/mnt/dedup/data/containers"],  "director_hostname":"baculadir.domain.com"}'

**Please note all the examples above can use a inventory file or --extra-vars or you may modify the playbooks to use any variable assignment allowed in Ansible.**

## Requirements

`jmespath` library needs to be installed on the host running the playbook. This is needed for the `json_query` filter.

To avoid `[WARNING]: Module remote_tmp ... did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the remote_tmp dir with the correct permissions
manually`, we do recommend to set the 'remote_tmp', for example, in the ansible.cfg:

remote_tmp = ~/.ansible/tmp

## Important Notes

All the roles will trigger the operating system update/upgrade as this is strongly recommended by Bacula Systems before installing Bacula Enterprise Edition.

## Licensing
License: BSD 2-Clause; see file LICENSE
