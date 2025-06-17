# Objectives
1. [Install Red Hat Ansible Engine on a control node](#install-red-hat-ansible-engine-on-a-control-node)
2. [Configure managed nodes](#configure-managed-nodes)
3. [Configure simple inventories](#configure-simple-inventories)
4. [Perform basic management of systems](#perform-basic-management-of-systems)
5. [Run a provided playbook against specified nodes](#run-a-provided-playbook-against-specified-nodes)

## Install Red Hat Ansible Engine on a control node

Install ansible using `dnf` or `python pip`
```bash
sudo dnf install pip
python -m pip install ansible
```
Check that ansible is installed:
```bash
ansible --version
```
Output:
```
ansible [core 2.18.6]
  config file = None
  configured module search path = ['/home/ec2-user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/ec2-user/.local/lib/python3.12/site-packages/ansible
  ansible collection location = /home/ec2-user/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/ec2-user/.local/bin/ansible
  python version = 3.12.9 (main, Mar 31 2025, 00:00:00) [GCC 14.2.1 20250110 (Red Hat 14.2.1-7)] (/usr/bin/python)
  jinja version = 3.1.5
  libyaml = True
```

## Configure managed nodes

Ensure that ssh keys are setup properly on control node
```bash
ssh-keygen -t rsa
```

Use ssh-copy-id to setup the access of managed node(s)
```bash
ssh-copy-id user@managed_host
```

## Configure simple inventories
`FILES` section under `man ansible`

```
FILES
       /etc/ansible/hosts -- Default inventory file

       /etc/ansible/ansible.cfg -- Config file, used if present

       ~/.ansible.cfg  --  User  config  file, overrides the default config if
       present

       ./ansible.cfg -- Local config file (in current working  directory)  as‐
       sumed to be 'project specific' and overrides the rest if present.

       As  mentioned above, the ANSIBLE_CONFIG environment variable will over‐
       ride all others.
```

Refer to `/etc/ansible/hosts` for sample:
```
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers:

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group:

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern, you can specify
# them like this:

## www[001:006].example.com

# You can also use ranges for multiple hosts: 

## db-[99:101]-node.example.com

# Ex 3: A collection of database servers in the 'dbservers' group:

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57


# Ex4: Multiple hosts arranged into groups such as 'Debian' and 'openSUSE':

## [Debian]
## alpha.example.org
## beta.example.org

## [openSUSE]
## green.example.com
## blue.example.com
```

## Perform basic management of systems

Test out ad-hoc mode from control node after initial setup and adding your inventory file:

```bash
ansible all -i hosts -m ping
```
For any issues encountered, you may add verbosity by using `-vvv` etc.

For playbooks, use `ansible-docs` command to see module usage examples.  For example, to see all `selinux` related modules, run the following command:

```bash
ansible-doc -l | grep -i selinux
```

Output:
```
ansible.posix.selinux                                                                                              ...
community.general.selinux_permissive                                                                               Ch...  
```

Then we can see the individual module and usage examples:

```bash
ansible-doc ansible.posix.selinux
```

Output:
```
.
.
.
EXAMPLES:
- name: Enable SELinux
  ansible.posix.selinux:
    policy: targeted
    state: enforcing

- name: Put SELinux in permissive mode, logging actions that would be blocked.
  ansible.posix.selinux:
    policy: targeted
    state: permissive

- name: Disable SELinux
  ansible.posix.selinux:
    state: disabled
.
.
.
```

Sample playbook that does the same thing as our ad-hoc command earlier:
```
---
- name: Example Playbook
  hosts: all

  tasks:
  - name: Ping hosts
    ping:
```

## Run a provided playbook against specified nodes

Run the playbook by using `ansible-playbook` command:
```bash
ansible-playbook -i hosts example
```

Output:
```
PLAY [Example Playbook] ***********************************************************************

TASK [Gathering Facts] ************************************************************************
[WARNING]: Platform linux on host XX.XX.XX.XX is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [XX.XX.XX.XX]

TASK [Ping hosts] *****************************************************************************
ok: [XX.XX.XX.XX]

PLAY RECAP ************************************************************************************
XX.XX.XX.XX             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
