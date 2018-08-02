Ansible Modules for Mikrotik RouterOS
=====================================

Overview
--------

This repository includes the following Ansible modules:
* ~~`routeros_facts`: Return `ansible_facts` for the device~~
* ~~`routeros_copy`: Copy files from the local machine to the device~~
* ~~`routeros_fetch`: Copy files from the device to the local machine~~
* `routeros`: Run a RouterOS command against the API directly
* ~~`routeros_item`: Create/update/delete configuration items in the API hierarchy~~

Requirements
------------

* Latest Ansible
* Latest [librouteros](https://pypi.python.org/pypi/librouteros)
* Mikrotik RouterOS-based device (e.g. RouterBOARD or Mikrotik CHR)
* API server enabled (check for port 8728 open)

Acknowledgements
----------------

I previously started by trying to update zahodi and senorsmile's
[ansible-mikrotik](https://github.com/zahodi/ansible-mikrotik) repository.

While trying to reorganize it, I decided it was ultimately better to start with
a collection of modules based more fundamentally on RouterOS's print/set/add
command set, and focus on flexibility rather than having a module for each
feature.

Python implementation of librouteros is not the best choice for api interaction.
There's a limitation in python librouteros: it doesn't accept any filter
arguments with the print command.
There is a [fork of librouteros](https://github.com/shuu01/librouteros),
which allows using filter arguments in print command like this:

```python
    from librouteros import connect

    api = connect(
        host='',
        username='',
        password='',
    )
    params =
    {
        '?type': 'vlan',
    }
    api(cmd='/interface/print', **params)
```

With this fork, ansible task will look like:

    tasks:
    - name: print interface with vlan type
      routeros:
        commands: /interface print ?type=vlan
        provider: "{{ api }}"
      register: result

With original version of librouteros one should use loops in tasks:

    tasks:
    - name: print interface
      routeros:
        commands: /interface print
        provider: "{{ api }}"
      register: result

    #vlans - empty list defined in inventory or role vars
    - name: get vlans
      set_fact:
        vlans: "{{ vlans + [item] }}"
      when: item.type == "vlan"
      loop: "{{ result.stdout.0 }}"

There are some examples in roles directory.

Usage
-----

    ansible-playbook playbook.yml -i hosts

License
-------

Code here is released under GPLv3 for maximum compatibility with Ansible Core.
