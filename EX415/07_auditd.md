# Objectives

Configure system auditing

1. [Write rules to log auditable events](#write-rules-to-log-auditable-events)
2. [Enable prepackaged rules](#enable-prepackaged-rules)
3. [Produce audit reports](#produce-audit-reports)

## TLDR
1. [auditd(8)](#auditd-overview)
2. auditd.conf(5)
3. auditctl(8) - look at the EXAMPLES section
4. ausearch(8)
5. aureport(8)

## auditd Overview
- `auditd` is started by `systemd`
- Log file location by default is at `/var/log/audit/audit.log`
- Rotation of log files:
```bash
service auditd rotate
```

### Packages
On RHEL, the audit system isn’t just one package:
- audit → base package (daemon + tools like auditctl, ausearch, aureport)
- audit-libs → libraries for user-space tools to interact with audit
- Optional packages:
  - audit-libs-python → Python bindings for audit (useful for custom scripts)
  - audispd-plugins → dispatcher plugins (e.g., send logs to syslog or remote server)

So, for basic logging and rules → only audit is required.  If you want remote logging or advanced integrations, install audispd-plugins.


### audit.log fields
Example:
```
type=SERVICE_STOP msg=audit(1757209312.247:1234): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=sshd comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'UID="root" AUID="unset"
```
Note: There maybe more fields depending on context. 
- type=: Type of audit record, e.g. USER_LOGIN, SYSCALL, EXECVE, etc 
- msg=audit(TIMESTAMP:EVENT_ID):
  - TIMESTAMP → seconds since the epoch + fractional seconds
  - EVENT_ID → unique event number for correlation (all records with same ID belong to the same high-level action)
  - timestamp in epoch, can convert using `date` command
- pid=: process ID
- uid=: user ID
- auid: audit user ID.  This stays constant for the user session.  Set at login time via pam_loginuid.so.  Special case of `-1` (4294967295) means it is not **unset**, typically for kernel threads or processes that does not originate from normal user login.
- ses=: audit session ID, unique per login session. Special case of `-1` means **unset / no session**
- subj=: security subject context of the process that triggered the audit event.  This is mostly used with Mandatory Access Control (MAC) frameworks like SELinux or AppArmor.
- msg=: inner msg is optional and free form text enclosed in single quotes `'`
- res=: res=success/fail → result of the action


## Write rules to log auditable events
- When writing rules, the order matters (first to fire).
- `auditctl` command affects runtime behaviour only. Add rules in `/etc/audit/audit.rules` for persistence.

### Configuration files
Configuration file locations:
- /etc/audit/
  - auditd.conf
  - audit.rules
  - rules.d/

### auditctl

`auditctl` options, shown by running `auditctl` without any options.

```
usage: auditctl [options]
    -a <l,a>                          Append rule to end of <l>ist with <a>ction
    -A <l,a>                          Add rule at beginning of <l>ist with <a>ction
    -b <backlog>                      Set max number of outstanding audit buffers
                                      allowed Default=64
    -c                                Continue through errors in rules
    -C f=f                            Compare collected fields if available:
                                      Field name, operator(=,!=), field name
    -d <l,a>                          Delete rule from <l>ist with <a>ction
                                      l=task,exit,user,exclude,filesystem
                                      a=never,always
    -D                                Delete all rules and watches
    -e [0..2]                         Set enabled flag
    -f [0..2]                         Set failure flag
                                      0=silent 1=printk 2=panic
    -F f=v                            Build rule: field name, operator(=,!=,<,>,<=,
                                      >=,&,&=) value
    -h                                Help
    -i                                Ignore errors when reading rules from file
    -k <key>                          Set filter key on audit rule
    -l                                List rules
    -m text                           Send a user-space message
    -p [r|w|x|a]                      Set permissions filter on watch
                                      r=read, w=write, x=execute, a=attribute
    -q <mount,subtree>                make subtree part of mount point's dir watches
    -r <rate>                         Set limit in messages/sec (0=none)
    -R <file>                         read rules from file
    -s                                Report status
    -S syscall                        Build rule: syscall name or number
    --signal <signal>                 Send the specified signal to the daemon
    -t                                Trim directory watches
    -v                                Version
    -w <path>                         Insert watch at <path>
    -W <path>                         Remove watch at <path>
    --loginuid-immutable              Make loginuids unchangeable once set
    --backlog_wait_time               Set the kernel backlog_wait_time
    --reset-lost                      Reset the lost record counter
    --reset_backlog_wait_time_actual  Reset the actual backlog wait time counter
```

### Enable / Disabling auditing:
```bash
auditctl -e 1      # enable auditing
auditctl -e 0      # disable auditing (not recommended)
```

### Listing and deleting rules
Show current loaded rules:
```bash
auditctl -l
```

Delete all current rules:
```bash
auditctl -D
```

### File and directory watches
Examples:
```bash
auditctl -w /etc/shadow -p wa # Note this slows the system
auditctl -a always,exit -F arch=b64 -F path=/etc/shadow -F perm=wa # Faster equivalent
auditctl -w /etc/shadow -p rw -k shadow_watch
```
- -w = watch a file or directory
- -p = permissions to monitor (read, write, xecute, attribute change)
- -k = key (tag) for easy searching later with ausearch -k

### System call rules
Log whenever a particular syscall is made:
```bash
auditctl -a always,exit -F arch=b64 -S execve -k exec_log
```
- -a always,exit → always audit on syscall exit
- -F arch=b64 → filter: 64-bit architecture
- -S execve → syscall to monitor (execve = process execution)
- -k exec_log → key for searching later

Another example: log attempts to delete files:
```bash
auditctl -a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat -k file_delete
```

### User-based rules
Restrict/audit by user ID:
```bash
auditctl -a always,exit -F uid=1000 -S execve -k user_exec
```

### Session & login tracking
Audit setuid system calls (privilege changes):
```bash
auditctl -a always,exit -F arch=b64 -S setuid,setgid -k identity
```

### Keystroke Logging
Enabling keystroke logging via `/etc/pam.d/system-auth`:
```
session required pam_tty_audit.so enable=root
```


## Enable prepackaged rules
- `/etc/audit/rules.d/` is where the current *active* rule file live. 
- Samples of predefined audit rules on RHEL can be found at:
  - `/usr/share/audit/sample-rules`


## Produce audit reports
`ausearch` command can be used to search for specific audit events.  `aureport` command can be used to generate report.  In both cases the `-i` option can be used to interpret  numeric  entities into text. For example, uid is converted to account name.