# Objectives

1. [Enable SELinux on a host running a simple application](#enable-selinux-on-a-host-running-a-simple-application)
2. [Interpret SELinux violations and determine remedial action](#interpret-selinux-violations-and-determine-remedial-action)
3. [Restrict user activity with SELinux user mappings](#restrict-user-activity-with-selinux-user-mappings)
4. [Analyze and correct existing SELinux configurations](#analyze-and-correct-existing-selinux-configurations)


## TLDR
1. [setenforce(8)](#setenforce8) - modify the mode SELinux is running in
2. [sestatus(8)](#sestatus) - SELinux status tool
3. [AUREPORT(8)](#aureport) - a tool that produces summary reports of audit daemon logs
4. semanage-user(8) - SELinux Policy Management SELinux User mapping tool
5. semange-login(8) - SELinux Policy Management linux user to SELinux User mapping tool
6. [sealert(8)](#sealert) - setroubleshoot client tool
7. audit2allow(1) - generate SELinux policy allow/dontaudit rules from logs of denied operations
8. semodule(8) - Manage SELinux policy modules
9. [semanage-fcontext(8)](#semanage-fcontext) - SELinux Policy Management file context tool
10. [restorecon(8)](#restorcon) - restore file(s) default SELinux security contexts
11. [semanage-port(8)](#semanage-port) - SELinux Policy Management port mapping tool
12. [setsebool(8)](#setsebool) - set SELinux boolean value



## Enable SELinux on a host running a simple application
System configuration file at `/etc/sysconfig/selinx`

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
#
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

### setenforce(8)
You can toggle between enforcing mode and permissive mode via `setenforce`. 
```
NAME
       setenforce - modify the mode SELinux is running in

SYNOPSIS
       setenforce [Enforcing|Permissive|1|0]

DESCRIPTION
       Use Enforcing or 1 to put SELinux in enforcing mode.
       Use Permissive or 0 to put SELinux in permissive mode.
```

### sestatus
To check current SELinux status, use `sestatus`:
```
       This  tool  is  used to get the status of a system running SELinux. It displays data about whether SELinux is enabled or dis‐
       abled, location of key directories, and the loaded policy with its status as shown in the example:
              > sestatus
              SELinux status:              enabled
              SELinuxfs mount:             /selinux
              SELinux root directory:      /etc/selinux
              Loaded policy name:          targeted
              Current mode:                permissive
              Mode from config file:       enforcing
              Policy MLS status:           enabled
              Policy deny_unknown status:  allow
              Memory protection checking:  actual (secure)
              Max kernel policy version:   26
```

### SELinux contexts
- user
- role
- type

`type` context is the most common. SELinux policies define the rules.  For most related commands like `ps`, `id`, `ls`, `netstat`, the `-Z` option can be used to show context information.

File contexts are by far the most common you will encounter.  Port contexts may also be encountered.

## Interpret SELinux violations and determine remedial action
Audit report can be examined for avc messages, which relates to SELinux events. 

### aureport
`aureport` can be used with the `-a` option:

```
$ aureport --help
usage: aureport [options]
	-a,--avc			Avc report
	-au,--auth			Authentication report
```

Alternatively, the `ausearch` command can be used:
```bash
$ ausearch -m avc
```

You can also `grep` on `/var/log/audit/audit.log`:
```bash
$ grep AVC /var/log/audit/audit.log
```

## Restrict user activity with SELinux user mappings
The `semanage user` commands is the SELinux Policy Management SELinux User mapping tool.  Refer to example section of the man page for usage:

```
EXAMPLE
       List SELinux users
       # semanage user -l
       Modify groups for staff_u user
       # semanage user -m -R "system_r unconfined_r staff_r" staff_u
       Assign user topsecret_u role staff_r and range s0-TopSecret
       # semanage user -a -R "staff_r" -rs0-TopSecret topsecret_u
```

The `semanage login` command is the SELinux Policy Management linux user to SELinux User mapping tool. Again refer to the EXAMPLES on how to use:
```
EXAMPLE
       Set the default SELinux user on the system to guest_u
       # semanage login -m -s guest_u __default__
       Map user gijoe to SELinux user staff_u and assign MLS range SystemLow-Secret
       # semanage login -a -s staff_u -rSystemLow-Secret gijoe
       Map all users in the engineering group to SELinux user staff_u
       # semanage login -a -s staff_u %engineering
```


## Analyze and correct existing SELinux configurations

### sealert
`grep sealert /var/log/messages` yields suggested troubleshooting steps using the `sealert` tool. The `sealert` tool gives suggestions on how to generate policies to allow violations.  Think through and use with care.  

Setting file contexts involves 2 steps. Setting the policy which defines the rules, and correcting any existing file / directory contexts.  

### semanage-fcontext
`semanage fcontext` is used to set the policy.  This does not retrospectively change existing contexts. `semanage fcontext -l` can be used to list records of the fcontext object type. 

Excerpt from `semange-fcontext` man page:

```
SYNOPSIS
       semanage fcontext [-h] [-n] [-N] [-S STORE] [ --add ( -t TYPE -f FTYPE -r RANGE -s SEUSER | -e EQUAL ) FILE_SPEC | --delete (
       -t TYPE -f FTYPE | -e EQUAL ) FILE_SPEC | --deleteall | --extract | --list [-C] | --modify ( -t TYPE -f  FTYPE  -r  RANGE  -s
       SEUSER | -e EQUAL ) FILE_SPEC ]
```

```
OPTIONS
       -a, --add
              Add a record of the specified object type

       -d, --delete
              Delete a record of the specified object type

       -l, --list
              List records of the specified object type

       -t TYPE, --type TYPE
              SELinux Type for the object
```

### restorcon

`restorecon` is the command to run to set correct the existing contexts.  Example is shown from `semanage-fcontext` man page:
```
EXAMPLE
       Remember to run restorecon after you set the file context
       Add file-context httpd_sys_content_t for everything under /web
       # semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
       # restorecon -R -v /web
```

### seinfo
If this tool is installed on the system, `seinfo -t` may also be used to examine context types.  

### semanage-port
`semanage port` is the SELinux Policy Management port mapping tool. Refer to the EXAMPLES section of the man page for example usage.  `-m` is used to modify an existing port definition. 

```
SYNOPSIS
       semanage  port  [-h]  [-n] [-N] [-S STORE] [ --add -t TYPE -p PROTOCOL -r RANGE port_name | port_range | --delete -p PROTOCOL
       port_name | port_range | --deleteall | --extract | --list [-C] | --modify -t TYPE -p PROTOCOL -r RANGE port_name | port_range
       ]
```

```
EXAMPLE
       List all port definitions
       # semanage port -l
       Allow Apache to listen on tcp port 81 (i.e. assign tcp port 81 label http_port_t, which apache is allowed to listen on)
       # semanage port -a -t http_port_t -p tcp 81
       Allow sshd to listen on tcp port 8991 (i.e. assign tcp port 8991 label ssh_port_t, which sshd is allowed to listen on)
       # semanage port -a -t ssh_port_t -p tcp 8991
```

### setsebool
`setsebool -P` option is used for persistence.

```
SYNOPSIS
       setsebool [ -PNV ] boolean value | bool1=val1 bool2=val2 ...

DESCRIPTION
       setsebool  sets  the current state of a particular SELinux boolean or a list of booleans to a given value. The value may be 1
       or true or on to enable the boolean, or 0 or false or off to disable it.

       Without the -P option, only the current boolean value is affected; the boot-time default settings are not changed.

       If the -P option is given, all pending values are written to the policy file on disk. So they will be persistent  across  re‐
       boots.

       If the -N option is given, the policy on disk is not reloaded into the kernel.

       If the -V option is given, verbose error messages will be printed from semanage libraries.

EXAMPLE
       Enable container_use_devices boolean (will return to persistent value after reboot)
       # setsebool container_use_devices 1
       Persistently enable samba_create_home_dirs and samba_enable_home_dirs booleans
       # setsebool -P samba_create_home_dirs=on samba_enable_home_dirs=on
```
