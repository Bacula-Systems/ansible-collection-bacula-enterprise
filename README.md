# Bacula Enterprise Collection for Ansible - baculasystems.bacula_enterprise

An Ansible Collection of roles to install one or more of the following Bacula Enterprise Edition components: A Director (using a PostgreSQL catalog), a File Daemon (FD) only, a Storage Daemon (SD) only, FD plugins, SD plugins, and/or BWeb in RHEL/CentOS/Debian/Ubuntu/OracleLinux/SLES Linux distributions.

## Ansible version compatibility

This collection has been tested against following Ansible versions: **>=2.9,<2.12**.

## Requirements

- A valid Bacula Systems subscription is required to make use of this Ansible Collection.

- The `jmespath` library package must be installed on the Ansible host running the playbook (eg: `apt install python3-jmespath` on Debian 10). This is needed for the `json_query` filter. This requirement is included in the requirements.txt file available in the collection.

- The `remote_tmp` variable should be set in your Ansible configuration to avoid the warning:

`[WARNING]: Module remote_tmp ... did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the remote_tmp dir with the correct permissions manually`

For example:

```
remote_tmp = ~/.ansible/tmp
```

## Important Notes

Bacula Systems strongly recommends having a fully patched system before installing any Bacula component. This being the case, all the roles are written to automatically trigger the Linux distribution's update/upgrade process (eg: apt update; apt upgrade in Debian/Ubuntu) prior to deploying any of the Bacula components.

Some roles have a `templates` directory with Clients, Jobs and Filesets that will be deployed when using the roles. We strongly recommend that you adapt these templates to your environment before deploying Clients and Plugins.

## Installing the Collection

Before using the Bacula Enterprise collection, it must be installed with the Ansible Galaxy command:

> \# ansible-galaxy collection install baculasystems.bacula_enterprise

## Roles

Role | Description
-------- | ---------------------
bee | install Bacula Enterprise Edition - PostgreSQL Catalog. This role may install PostgreSQL required packages if not present in the host. This role also installs the Bacula Director (DIR), a Storage Daemon (SD), and a File Daemon (SD).
bee-fdonly | install Bacula File Daemon only. This is a Bacula Client.
bee-sdonly | Install a Bacula Storage. This role follows the recommended procedure to install Bacula Enterprise Edition and stop/disable the Director.
bee-fdplugin | install a Bacula Enterprise Plugin in a Bacula Client host. Please note it is required to have a Bacula File Daemon installed and running before installing the File Daemon Plugin.
bee-sdplugin | install a Bacula Enterprise Plugin in a Bacula Storage host. Please note it is required to have a Bacula Storage Daemon installed and running before installing the Storage Daemon Plugin.
bee_plugindependencies | install Plugins required dependencies. This role is required and used by both the *bee-fdplugin* and the *bee-sdplugin* roles.

### Roles Variables

The roles have most of the variables defined in the `vars` directories or in the `default/main.yml` files.

Please set your Bacula Enterprise Edition subscription's repository URL (download area), the `dl_area` variable, in the `default/main.yml` file for each role prior to using them.

You may use the following command to set all `dl_area` values at once:

> \# find /root/.ansible/collections/ansible_collections/baculasystems/bacula_enterprise/roles/ -name main.yml -exec sed -i 's/@@customer@/<your_download_area_string>/g' {} \\;

The Bacula Clients (File Daemons) and the Bacula Storage Daemons will use the password used by the Director {} resource present in the bacula-fd.conf and bacula-sd.conf files respectively to communicate with the remote Director provided in the deployment.

The variables in the following table must be set in a playbook, in an inventory file, or by passing them with the `--extra-vars` option on the command line. Depending on the role used, the following variables may be required: `bee_version, director_hostname, client_hostname, client_name`, `storage_hostname`, `storage_name`, `volumes_directory` and `dedup_directories`


Variable | Description
-------- | ---------------------
bee_version | the Bacula Enterprise Edition version, for example: `16.0.7`
director_hostname | the FQDN of the Director host, for example: `baculadir.example.com`. The name of the Director is, by default, the hostname + "-dir". In this example, the Director name will be `baculadir-dir`
client_hostname | the FQDN of the Client host, for example, `baculaclient.example.com`
client_name | the name of the File Daemon/Client. For example, `baculaclient1-fd`. If not specified, the `client_name` will be automatically created using the `client_hostname` + `-fd`
storage_hostname | the FQDN of the Storage host, for example, `baculasd1.example.com`
storage_name | the name of the Storage. For example, `baculasd1-sd`. If not specified, the `storage_name` will be the `storage_hostname` + `-sd`
volumes_directory | this variable will be required when using the `bee-sdonly` and `bee-sdplugin` roles. This is where the Bacula volumes will be stored. It is the value used in the SD Device resource(s) `Archive Device = xxxx` directive(s).
dedup_directories | these are the Dedup Index, the Dedup Containers and the Dedup Volumes directories required when using the `bee-sdplugin` role to install the GED plugin. If not defined, the default values are `/opt/bacula/dedup/index`, `/opt/bacula/dedup/containers` and `/opt/bcula/dedup/volumes`. They may be modified using `--extra-vars='{"dedup_directories": [/mnt/dedup/index, /mnt/dedup/containers, /mnt/dedup/volumes]}'`. Please note the first directory will be used for the Dedup Index and the second one for the Dedup Containers.
catalog_name | by default, the roles will use the first Catalog name found in the configuration files. If you wish, you may specify a different Catalog name to be used for the Clients installed using the `catalog_name` variable.
jobdefs_name | by default, the roles will use the first JobDefs resource name found in the configuration files. If you wish, you may specify a different JobDefs resource name to be used for the Jobs installed using the `jobdefs_name` variable.
install_gdb | this variable allows you to install, along with Bacula Enterprise, the gdb debugger. Having gdb installed by default in the Bacula Enterprise host is helpful as it provides helpful information for debugging purposes. The default is `no`.

### Usage and Examples

The following examples use the playbooks in the `tests` directory of the Bacula Enterprise collection.

1) To deploy Bacula Enterprise Edition (DIR, SD, and FD) `16.0.7` on a host with the FQDN `baculadir.example.com` using the --extra_vars command line option, and the `bee.yml` playbook:

> \# ansible-playbook -i baculadir.example.com, tests/bee.yml --extra-vars "director_hostname=baculadir.example.com bee_version=16.0.7"

This is the `tests/bee.yml` file referenced in the command line above:
```
---
- hosts: "{{ director_hostname }}"
  remote_user: root
  tasks:
  - name: Install Bacula Enterprise Edition - PostgreSQL Catalog
    import_role:
      name: baculasystems.bacula_enterprise.bee
```

2) Using an inventory file to install two remote Bacula Enterprise Edition version `16.0.7` File Daemon to the hosts `client1.example.com` and `client2.example.com`, and deploy the two clients' resource definitions in the Director's configuration on the `baculadir.example.com` host, first an inventory file needs to be created:

Note: In this example, please be sure to edit the `tests/clients.yaml` inventory file to match your environment.

The `tests/clients.yaml` inventory file (edit to match systems in your environment):
```
---
clients:
  hosts:
    client1.example.com:
      client_hostname: client1.example.com
      client_name: client1-fd
    client2.example.com:
      client_hostname: client2.example.com

  vars:
    bee_version: 16.0.7
    director_hostname: baculadir.example.com
```

Then, to deploy the two Bacula File Daemons to the two hosts using the `bee-fdonly.yml` playbook and the above `clients.yaml` inventory file:

> \# ansible-playbook -i tests/clients.yaml tests/bee-fdonly.yml

This is the `tests/bee-fdonly.yml` playbook referenced in the command line above:
```
---
- hosts: clients
  tasks:

  - name: Install and configure Bacula File Daemon
    import_role:
      name: baculasystems.bacula_enterprise.bee_fdonly
```

3) To install the Bacula Enterprise Edition MySQL File Daemon Plugin version `16.0.7` in the remote client on the `client1.example.com` host, and deploy the plugin configuration (basic FileSet, not including plugin options in most of the cases) in the Director's configuration on the `baculadir.example.com` host, using the --extra_vars command line option:

> \# ansible-playbook tests/bee-fdplugin.yml -i client1.example.com, --extra-vars "client_hostname=client1.example.com client_name=client1-fd bee_version=16.0.7 fdplugin=mysql director_hostname=baculadir.example.com"

In the above example, the client name in the bacula-fd.conf file will be modified to `client1-fd` instead of `client1.example.com-fd`. The name used in the Client resource in the Director configuration will also be deployed using `client1-fd`.

This is the `tests/bee-fdplugin.yml` playbook referenced in the commmand line above:
```
---
- hosts: "{{ client_hostname }}"
  tasks:
  - name: Install File Daemon, if not installed
    import_role:
      name: baculasystems.bacula_enterprise.bee_fdonly
  - name: Install Plugin dependencies
    import_role:
      name: baculasystems.bacula_enterprise.bee_plugindependencies
  - name: Install the File Daemon Plugin
    import_role:
      name: baculasystems.bacula_enterprise.bee_fdplugin
```

4) In the example, two remote Bacula Enterprise Edition version `16.0.7` Storage Daemons will be deployed to the hosts `storage1.example.com` and `storage2.example.com`, using an inventory file. Bacula Enterprise Edition DIR, SD, and FD components will also be installed and the Bacula Director service will be automatically disabled. First an inventory file needs to be created:

The `tests/storages.yaml` inventory file (edit to match systems in your environment):
```
---
storages:
  hosts:
    storage1.example.com:
      storage_hostname: storage1.example.com
    storage2.example.com:
      storage_hostname: storage2.example.com
  vars:
    director_hostname: baculadir.example.com
    bee_version: 16.0.7
    volumes_directory: /opt/bacula/volumes
```

Then, to deploy the two Bacula Storage Daemons to the two hosts using the `tests/bee-sdonly.yml` playbook and the above `tests/storages.yaml` inventory file:

> \# ansible-playbook -i tests/storages.yaml tests/bee-sdonly.yml

This is the `tests/bee-sdonly.yml` playbook referenced in the command line above:
```
---
- hosts: storages
  tasks:

  - name: Install and configure Bacula Storage Daemon only
    import_role:
      name: baculasystems.bacula_enterprise.bee_sdonly
```

5) To install a Bacula Enterprise Edition dedup (GED) Storage Daemon Plugin version `16.0.7` on the `storage1.example.com` host (deployed in the example 4 above), using the --extra_vars command line option:

> \# ansible-playbook -i storage1.example.com, tests/bee-sdplugin.yml --extra-vars '{"storage_hostname":"storage1.example.com",  "storage_name":"bacula-dedup-sd",  "bee_version":"16.0.7", "sdplugin":"dedup",  "volumes_directory":"/mnt/dedup/data/volumes",  "dedup_directories":["/mnt/dedup/index", "/mnt/dedup/data/containers"],  "director_hostname":"baculadir.example.com"}'

This is the `tests/bee-sdplugin.yml` playbook referenced in the command line above:
```
---
- hosts: "{{ storage_hostname }}"
  tasks:
  - name: Install Plugin Dependencies
    import_role:
      name: baculasystems.bacula_enterprise.bee_plugindependencies
  - name: Install the Storage Daemon Plugin
    import_role:
      name: baculasystems.bacula_enterprise.bee_sdplugin
```

6) To deploy Bacula Enterprise BWeb Management Suite  `16.0.7` on a host with the FQDN `baculadir.example.com` using the --extra_vars command line option, and the `bweb.yml` playbook:

> \# ansible-playbook -i baculadir.example.com, tests/bweb.yml --extra-vars "director_hostname=baculadir.example.com bee_version=16.0.7"

This is the `tests/bee.yml` file referenced in the command line above:
```
---
- hosts: "{{ director_hostname }}"
  tasks:
  - name: Install BWeb Management Suite
    import_role:
      name: baculasystems.bacula_enterprise.bweb
```

**From Bacula Enterprise Edition 16.0.7, it is possible to use the Bacula Enterprise Ansible Collection to deploy new File Daemons and/or new Storage Daemon in your environment even if you have the configuration files already split using BWeb. In this collection, we provide the re-split-configuration.yml playbook that will re-split the configuration added by the Ansible collection for the new resources in the current BWeb directory structure:**

> \# ansible-playbook -i baculadir.example.com, tests/re-split-configuration.yml --extra-vars "director_hostname=baculadir.example.com"

```
---
- hosts: "{{ director_hostname }}"
  remote_user: root
  tasks:
  - name: Re split the configuration using the /opt/bweb/bin/split_bacula_conf.pl script
    shell: "/opt/bweb/bin/split_bacula_conf.pl -F -d /opt/bacula/etc/conf.d/ -b /opt/bacula/bin/ -f /opt/bacula/etc/bacula-dir.conf -c Director"
    become: yes
    become_user: bacula

  - name: reload Director configuration
    shell: echo "reload" | /opt/bacula/bin/bconsole
    register: reload_result
    delegate_to: "{{ director_hostname }}"
  - debug:  msg="{{ reload_result.stdout.split('\n') }}"
  - debug:  msg="{{ reload_result.stderr.split('\n') }}"

  - name: display any message in bconsole
    shell: echo "messages" | /opt/bacula/bin/bconsole
    register: status_messages
    delegate_to: "{{ director_hostname }}"
  - debug:  msg="{{ status_messages.stdout.split('\n') }}"
```

**Please note all the examples above can use an inventory file and/or --extra-vars or you may modify the playbooks to use any variable assignment allowed in Ansible.**


## Licensing

License: Copyright (C) 2000-2020 Kern Sibbald / BSD 2-Clause; see file [LICENSE](./LICENSE)
