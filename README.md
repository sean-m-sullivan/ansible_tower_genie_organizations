# ansible_tower_genie_organizations
## Description
An Ansible Role to create Organizations in Ansible Tower.

## Requirements 
ansible-galaxy collection install -r tests/collections/requirements.yml to be installed 
Currently:
  awx.awx
## Variables
|Variable Name|Default Value|Required|Description|Example|
|:---:|:---:|:---:|:---:|:---:|
|`tower_url`|""|yes|URL to the Ansible Tower Server.|127.0.0.1|
|`tower_verify_ssl`|False|no|Whether or not to validate the Ansible Tower Server's SSL certificate.||
|`tower_oauthtoken`|""|yes|Tower Admin User's token on the Ansible Tower Server.  This should be stored in an Ansible Vault at or elsewhere and called from a parent playbook.||
|`organizations`|"see below"|yes|Data structure describing your orgainzation or orgainzations Described below.||

### Secure Logging Variables
The following Variables compliment each other. 
If Both variables are not set, secure logging defaults to false. 
The role defaults to False as normally the add organization task does not include sensative information. 
tower_genie_organizations_secure_logging defaults to the value of tower_genie_secure_logging if it is not explicitly called. This allows for secure logging to be toggled for the entire suite of genie roles with a single variable, or for the user to selectively use it. 

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`tower_genie_organizations_secure_logging`|False|no|Whether or not to include the sensative Organization role tasks in the log.  Set this value to `True` if you will be providing your sensitive values from elsewhere.|
|`tower_genie_secure_logging`|False|no|This variable enables secure logging as well, but is shared accross multiple roles, see above.|

## Data Structure
### Varibles
|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`name`|""|yes|Name of Organization|
|`description`|False|no|Description of  of Organization.|
|`custom_virtualenv`|""|no|Local absolute file path containing a custom Python virtualenv to use.|
|`max_hosts`|""|no|The max hosts allowed in this organizations.|

### Standard Organization Structure
#### Json Example
```json
---
{
    "organizations": [
      {
        "name": "Default",
        "description": "This is the Default Group"
      },
      {
        "name": "Automation Group",
        "description": "This is the Automation Group",
        "custom_virtualenv": "/opt/cust/enviroment/",
        "max_hosts": 10
      }      
    ]
}
```
#### Ymal Example
```yaml
---
organizations:
- name: Default
  description: This is the Default Group
- name: Automation Group
  description: This is the Automation Group
  custom_virtualenv: "/opt/cust/enviroment/"
  max_hosts: 10
```
## Playbook Examples
### Standard Role Usage
```yaml
---

- name: Add Organizations to Tower
  hosts: localhost
  connection: local
  gather_facts: false

#Bring in vaulted Ansible Tower secrets
  vars_files:
    - ../tests/vars/tower_secrets.yml

  tasks:

    - name: Get token for use during play
      uri:
        url: "https://{{ tower_server }}/api/v2/tokens/"
        method: POST
        user: "{{ tower_username }}"
        password: "{{ tower_passname }}"
        force_basic_auth: yes
        status_code: 201
        validate_certs: no
      register: user_token
      no_log: True

    - name: Set Tower oath Token
      set_fact:
        tower_oauthtoken: "{{ user_token.json.token }}"

    - name: Import JSON
      include_vars:
        file: "json/organizations.json"
        name: organizations_json

    - name: Add Organizations
      include_role: 
        name: ../..
      vars:
        organizations: "{{ organizations_json.organizations }}"
```
## License
[MIT](LICENSE)

## Author
[Andrew J. Huffman](https://github.com/ahuffman)
