Ansible Role: oracle-manage-dbstate
===================================

Manage Oracle Database state (startup/shutdown).


Supported OS:
-------------
* RedHat
* CentOS
* OracleLinux


Requirements
------------

This role uses `oracle` role.

It requires following variables to be defined:
* `oracle_manage_dbstate_oracle_home` -> ORACLE_HOME
* `oracle_manage_dbstate_oracle_sid` -> ORACLE_SID
* `oracle_manage_dbstate_command` -> startup/shutdown command along with option


Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # ORACLE_HOME
    oracle_manage_dbstate_oracle_home:

    # ORACLE_SID
    oracle_manage_dbstate_oracle_sid:

    # startup/shutdown command to execute
    oracle_manage_dbstate_command:     # see 'oracle_manage_dbstate_commands'

    # startup/shutdown acceptable commands
    oracle_manage_dbstate_commands:
      - 'SHUTDOWN IMMEDIATE'
      - 'STARTUP'
      - 'STARTUP NOMOUNT'
      - 'STARTUP MOUNT'
      - 'STARTUP RESTRICT'
      - 'STARTUP MOUNT RESTRICT'
      - 'STARTUP UPGRADE'

    # assert that startup/shutdown command is valid
    oracle_manage_dbstate_verify_command: true

    # print sqlplus command output
    oracle_manage_dbstate_print_command_output: false

    # handle sqlplus errors, i.e. execute 'whenever sqlerror exit 1'
    oracle_manage_dbstate_handle_sqlplus_errors: true

    # sqlplus error handling command
    oracle_manage_dbstate_sqlplus_sqlerror_command: "{% if oracle_manage_dbstate_handle_sqlplus_errors %}whenever sqlerror exit 1{% else %}{% endif %}"


Example Playbook
----------------

    - name: Manage Oracle Database state (mount databases)
      hosts: ora-servers
      gather_facts: true
      become: true
      become_user: '{{ oracle_user }}'

      tasks:

      - import_role:
          name: oracle-gatherinfo-gi
        tags:
          - oracle-gatherinfo-gi

      - import_role:
          name: oracle-gatherinfo-databases
        tags:
          - oracle-gatherinfo-databases

      - include_role:
          name: oracle-manage-dbstate
        vars:
          oracle_manage_dbstate_oracle_home: '{{ _oracle_homes_backup_outer_item.oracle_home }}'
          oracle_manage_dbstate_oracle_sid: '{{ _oracle_homes_backup_outer_item.instance_name }}'
          oracle_manage_dbstate_command: "STARTUP MOUNT"
          oracle_manage_dbstate_handle_sqlplus_errors: true
        with_items:
          - "{{ oracle_databases }}"
        loop_control:
          label: "[ORACLE_HOME: {{ _oracle_homes_backup_outer_item.oracle_home }}]"
          loop_var: _oracle_homes_backup_outer_item
        when: _oracle_homes_backup_outer_item.instance_name == 'ORCL'
        tags:
          - oracle-manage-dbstate


Inside `vars/main.yml` or `group_vars/..` or `host_vars/..`:


    #-------------------------------------------------
    # overrides role 'oracle-manage-dbstate' variables
    #-------------------------------------------------

    # print sqlplus command output
    oracle_manage_dbstate_print_command_output: true

    # handle sqlplus errors, i.e. execute 'whenever sqlerror exit 1'
    oracle_manage_dbstate_handle_sqlplus_errors: false

    
License
-------

GPLv3 - GNU General Public License v3.0

Author Information
------------------

This role was created in 2018 by [Krzysztof Lewandowski](mailto:Krzysztof.Lewandowski@fastmail.fm).

