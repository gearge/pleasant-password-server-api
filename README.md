Pleasant Password Server API
============================

Ansible role: Wrapper for Pleasant Password Server (PPS) API, [currently v4 only](https://pleasantpasswords.com/info/pleasant-password-server/m-programmatic-access/restful-api#v4).

Requirements
------------

Requires a running instance of Pleasant Password Server 7.0 or higher. See https://pleasantpasswords.com/info/pleasant-password-server

Role Variables
--------------

Reads

| Variable name         | Default value (if any) | Description |
| :-------------------- | :--------------------- | :---------- |
| `pps_oauth2_username` | `admin`                | User username to acquire an authorization token. See https://pleasantpasswords.com/info/pleasant-password-server/m-programmatic-access/restful-api#v4 |
| `pps_oauth2_password` | `admin`                | User password to acquire an authorization token. See https://pleasantpasswords.com/info/pleasant-password-server/m-programmatic-access/restful-api#v4 |
| `pps_oauth2_token`    |                        | Obtained by [tasks/acquire-oauth2-token.yml](tasks/acquire-oauth2-token.yml) using `pps_oauth2_username` and `pps_oauth2_password`. **This var is NOT settable!** |
| `pps_api_version`     | `v4`                   | PPS API version. **ONLY API v4 is currently supported!** |
| `pps_authority`       | `localhost:10001`      | `[domain]:[port]` of PPS. Will be used as `https://{pps_authority}/api/{pps_api_version}/rest/` |
| `pps_operation`       |                        | One of `create-credentialgroup`, `create-credential`, `delete-credentialgroup`, `delete-credential`, `read-credentialgroup-root`, `read-credentialgroup`, `read-credential`, `search`, `update-credentialgroup`, `update-credential`, `create-or-update-entry`, `search-and-filter-entries`, `search-filter-and-delete-entry`, `create-or-update-folder` |
| `pps_credential`      |                        | Required when `pps_operation` is `create-or-update-entry`, `search-filter-and-delete-entry`, `create-credential`, `delete-credential`, `read-credential`, `update-credential`. See [tasks/convenience/create-or-update-entry.yml](tasks/convenience/create-or-update-entry.yml), [tasks/convenience/search-filter-and-delete-entry.yml](tasks/convenience/search-filter-and-delete-entry.yml), [tasks/v4/create-credential.yml](tasks/v4/create-credential.yml), [tasks/v4/delete-credential.yml](tasks/v4/delete-credential.yml), [tasks/v4/read-credential.yml](tasks/v4/read-credential.yml), [tasks/v4/update-credential.yml](tasks/v4/update-credential.yml) |
| `pps_credentialgroup` |                        | Required when `pps_operation` is `create-or-update-folder`, `create-credentialgroup`, `delete-credentialgroup`, `read-credentialgroup`, `update-credentialgroup`. See [tasks/convenience/create-or-update-folder.yml](tasks/convenience/create-or-update-folder.yml), [tasks/v4/create-credentialgroup.yml](tasks/v4/create-credentialgroup.yml), [tasks/v4/delete-credentialgroup.yml](tasks/v4/delete-credentialgroup.yml), [tasks/v4/read-credentialgroup.yml](tasks/v4/read-credentialgroup.yml), [tasks/v4/update-credentialgroup.yml](tasks/v4/update-credentialgroup.yml) |
| `pps_search`          |                        | Required when `pps_operation` is `search-and-filter-entries`. See [tasks/convenience/search-and-filter-entries.yml](tasks/convenience/search-and-filter-entries.yml) |
| `pps_query`           |                        | Required when `pps_operation` is `search`. See [tasks/v4/search.yml](tasks/v4/search.yml) |

Sets

| For `pps_operation`              | Result variable |
| :------------------------------- | :-------------- |
| `create-credentialgroup`         | `pps_create_credentialgroup_result` |
| `create-credential`              | `pps_create_credential_result` |
| `delete-credentialgroup`         | `pps_delete_credentialgroup_result` |
| `delete-credential`              | `pps_delete_credential_result` |
| `read-credentialgroup-root`      | `pps_read_credentialgroup_root_result` |
| `read-credentialgroup`           | `pps_read_credentialgroup_results` |
| `read-credential`                | `pps_read_credential_results` |
| `search`                         | `pps_search_results` |
| `update-credentialgroup`         | `pps_update_credentialgroup_result` |
| `update-credential`              | `pps_update_credential_result` |
| `create-or-update-entry`         | `pps_create_or_update_entry_result` |
| `search-and-filter-entries`      | `pps_search_and_filter_entries_results` |
| `search-filter-and-delete-entry` | `pps_search_filter_and_delete_entry_result` |
| `create-or-update-folder`        | `pps_create_or_update_folder_result` |

Dependencies
------------

This role doesn't depend on other roles hosted on Ansible Galaxy.

Example Playbooks
-----------------

Note: Here Ansible Vault file (`vault.yml`) is used to provide values for secrets, with `vault_pps_oauth2_username` defined within the vault mapped to `pps_oauth2_username`, `vault_pps_oauth2_password` mapped to `pps_oauth2_password`, etc. Read [man page for `ansible-vault`](https://docs.ansible.com/ansible/2.9/cli/ansible-vault.html) for how to create this file.

`git clone https://github.com/gearge/pleasant-password-server-api.git pps-api`

Search for entries containing string "FOO", next select only those with Name == "FOO" under path "Root/PATH/TO/" on PPS:

    ---
    - hosts: localhost
      gather_facts: yes # switched on for ansible_date_time
      connection: local
      #no_log: true
      vars:
        pps_authority:       "{{ vault_pps_authority }}"
        pps_api_version:     "{{ vault_pps_api_version }}"
        pps_oauth2_username: "{{ vault_pps_oauth2_username }}"
        pps_oauth2_password: "{{ vault_pps_oauth2_password }}"
        pps_operation:       "search-and-filter-entries"
        pps_search:
          query: "FOO"
          filters:
            Name: "FOO"
            FullPath: "Root/PATH/TO/"
      vars_files:
        - vault.yml
      roles:
        - pps-api
      post_tasks:
        - debug: var=pps_search_and_filter_entries_results

Create a new or update an existing entry with Name == "FOO" under path "Root/PATH/TO/" on PPS:

    ---
    - hosts: localhost
      gather_facts: yes # switched on for ansible_date_time
      connection: local
      #no_log: true
      vars:
        pps_authority:       "{{ vault_pps_authority }}"
        pps_api_version:     "{{ vault_pps_api_version }}"
        pps_oauth2_username: "{{ vault_pps_oauth2_username }}"
        pps_oauth2_password: "{{ vault_pps_oauth2_password }}"
        pps_operation:       "create-or-update-entry"
        pps_credential:
          Name: "BAR TEST 001"
          Username: "USERNAME"
          Password: "NEWPASSWORD"
          FullPath: "Root/PATH/TO/" 
          Tags: [{'Name': 'BAR'}]
          Url: "https://my.example.com/"
          Notes: "TEST"
      vars_files:
        - vault.yml
      roles:
        - pps-api
      post_tasks:
        - debug: var=pps_create_or_update_entry_result

License
-------

The MIT License (MIT)

Giorgi Tavkelishvili
------------------

