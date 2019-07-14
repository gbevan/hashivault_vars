# Ansible Vars Plugin for Hashicorp Vault


|![](https://img.shields.io/pypi/v/hashivault-vars.svg)|![](https://img.shields.io/pypi/status/hashivault-vars.svg)|![](https://img.shields.io/pypi/format/hashivault-vars.svg)|![](https://img.shields.io/pypi/l/hashivault-vars.svg)|![](https://travis-ci.com/goethite/hashivault_vars.svg?branch=master)|
|-|-|-|-|-|

An Ansible Vars Plugin for Hashicorp Vault to lookup credentials/secrets,
injecting these into the playbook run (e.g. `ansible_user`, `ansible_password`,
etc).

Use Hashicorp Vault like you would ansible-vault'ed group_vars,
domain_vars [a new concept in this module!] and host_vars.

This module was originaly developed for the [gostint](https://goethite.github.io/gostint/)
project.

## Prereqs
* Ansible
* You may need `pip install urllib3`
* `pip install hvac`

## Installation

```bash
sudo pip install hashivault-vars
```

## Enable in Ansible
In `ansible.cfg`:
```
vars_plugins = /usr/local/lib/python2.7/dist-packages/hashivault_vars
```

Or, symlink from ansible's vars plugins folder to `hashivault_vars.py`, e.g.:
```bash
$ cd /usr/local/lib/python2.7/dist-packages/ansible/plugins/vars
$ sudo ln -s /usr/local/lib/python2.7/dist-packages/hashivault_vars/hashivault_vars.py .
```

On Alpine Linux:
```bash
pip install hvac hashivault-vars && \
ln -s /usr/lib/python2.7/site-packages/hashivault_vars/hashivault_vars.py \
  /usr/lib/python2.7/site-packages/ansible/plugins/vars
```

## Vault Secret Paths
Root path in vault:

* `/secret/ansible/`

Precendence (applied top to bottom, so last takes precendence):
* Groups:
  * `/secret/ansible/groups/all`
  * `/secret/ansible/groups/ungrouped`
  * `/secret/ansible/groups/your_inv_item_group`
  * ...

* Hosts/Domains:
  * `/secret/ansible/domains/com`
  * `/secret/ansible/{connection}/domains/com`
  * `/secret/ansible/domains/example.com`
  * `/secret/ansible/{connection}/domains/example.com`
  * `/secret/ansible/hosts/hosta.example.com`
  * `/secret/ansible/{connection}/hosts/hosta.example.com`

where `{connection}` is `ansible_connection`, e.g.: "ssh", "winrm", ...
(this plugin attempts to make assumptions where `ansible_connection` is not
set, but does not assume to inject this into vars in the playbook. Best
practice therefore would be to set `ansible_connection` in your ansible
inventory).

All values retrieved from these paths are mapped as ansible variables,
e.g. `ansible_user`, `ansible_password`, etc.

The layered lookups are merged, with the last taking precedence over
earlier lookups.

Lookups to the vault are cached for the run.

## Developer Notes

### Travis CI
Pull requests and merges to master trigger pylint and BATS tests.

### Running BATS tests
in vagrant:
```bash
$ tests/test.sh
```

### Enable Debugging
(danger, will reveal retrieved vault secrets in the ansible log)

Set environment variable `HASHIVAULT_VARS_DEBUG=1`.

### Release to PyPi
From vagrant (pip prereqs are required), e.g.:
```bash
$ ./setup.py sdist bdist_wheel
```

Release from host:
```bash
$ twine upload dist/hashivault_vars-0.1.17*
```

### Ansible Deprecation Warnings

#### TRANSFORM_INVALID_GROUP_CHARS
```
ansible-playbook 2.8.2
  config file = /home/vagrant/src/tests/bats/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.6/dist-packages/ansible
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.6.8 (default, Jan 14 2019, 11:02:34) [GCC 8.0.1 20180414 (experimental) [trunk revision 259383]]
Using /home/vagrant/src/tests/bats/ansible.cfg as config file
setting up inventory plugins
host_list declined parsing /home/vagrant/src/tests/bats/0100_hosts as it did not pass it's verify_file() method
script declined parsing /home/vagrant/src/tests/bats/0100_hosts as it did not pass it's verify_file() method
auto declined parsing /home/vagrant/src/tests/bats/0100_hosts as it did not pass it's verify_file() method
Set default localhost to 127.0.0.1
Not replacing invalid character(s) "{'.'}" in group name (my.com)
[DEPRECATION WARNING]: The TRANSFORM_INVALID_GROUP_CHARS settings is set to allow bad characters in group names by default, this will change, but still be user
configurable on deprecation. This feature will be removed in version 2.10. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
 [WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
Not replacing invalid character(s) "{'.'}" in group name (my.com)
```
By default from ansible 2.10 FQDNs/Domains (with .'s) as group names will not be
supported, and may need to be entered into vault with _'s instead of .'s...
