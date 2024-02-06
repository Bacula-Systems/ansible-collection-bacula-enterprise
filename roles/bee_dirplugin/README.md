# Bacula Enterprise Director Plugin
This role will install or update a Bacula Director Plugin on linux platforms supported by the Bacula Enterprise Edition software. Additionally, if specified, it will deploy the 

## Role Variables

Basically, the user download area is required to download and install the packages or the URL to a local repository. 

dl_area: https://qa.baculasystems.com/dl/*user_download_area*

These variables can be added in the defaults/main.yml files for default role variables.

Specific variables for the repository definition to be used in the role are placed in individual files per platform in the role vars directory.

Additional required variables:

bee_version: 12.6.0


## Example Playbook

---
- hosts: "{{ vm_name }}.supportlab.lan"
  remote_user: root
  tasks:
  - import_role:
     name: bee-dirplugin

### Usage

ansible-playbook /path/to/playbook/bee-dirplugin.yml --extra-vars "bee_version=12.6.0 build=release  vm_name=baculadir"
