# Objectives

Restrict USB devices

1. [Install USBGuard](#install-usbguard)
2. [Write device policy rules with specific criteria to manage devices](#write-device-policy-rules-with-specific-criteria-to-manage-devices)
3. [Manage administrative policy and daemon configuration](#manage-administrative-policy-and-daemon-configuration)

## TLDR
1. usbguard(1)
2. usbguard-daemon(8)
3. usbguard-daemon.conf(5)
4. usbguard-rules.conf(5)
5. /usr/share/hwdata/usb.ids

```
[user@localhost /etc]$ usbguard --help
 Usage: usbguard [OPTIONS] <command> [COMMAND OPTIONS] ...

 Options:

 Commands:
  get-parameter <name>           Get the value of a runtime parameter.
  set-parameter <name> <value>   Set the value of a runtime parameter.
  list-devices                   List all USB devices recognized by the USBGuard daemon.
  allow-device <id|rule|p-rule>  Authorize a device to interact with the system.
  block-device <id|rule|p-rule>  Deauthorize a device.
  reject-device <id|rule|p-rule> Deauthorize and remove a device from the system.

  list-rules                     List the rule set (policy) used by the USBGuard daemon.
  append-rule <rule>             Append a rule to the rule set.
  remove-rule <id>               Remove a rule from the rule set.

  generate-policy                Generate a rule set (policy) based on the connected USB devices.
  watch                          Watch for IPC interface events and print them to stdout.
  read-descriptor                Read a USB descriptor from a file and print it in human-readable form.

  add-user <name>                Add USBGuard IPC user/group (requires root privilges)
  remove-user <name>             Remove USBGuard IPC user/group (requires root privileges)
```


## Install USBGuard
Installation is done by using dnf command `sudo dnf install usbguard`.

```bash
[user@localhost ~]$ sudo dnf install usbguard
Updating Subscription Management repositories.
.
.
.
Dependencies resolved.
================================================================================
 Package          Arch   Version         Repository                        Size
================================================================================
Installing:
 usbguard         x86_64 1.0.0-16.el9    rhel-9-for-x86_64-appstream-rpms 473 k
Installing dependencies:
 libqb            x86_64 2.0.8-1.el9     rhel-9-for-x86_64-appstream-rpms  95 k
 protobuf         x86_64 3.14.0-16.el9   rhel-9-for-x86_64-appstream-rpms 1.0 M
Installing weak dependencies:
 usbguard-selinux noarch 1.0.0-16.el9    rhel-9-for-x86_64-appstream-rpms  23 k

Transaction Summary
================================================================================
Install  4 Packages

Total download size: 1.6 M
Installed size: 5.1 M
Is this ok [y/N]: y
.
.
.

Installed:
  libqb-2.0.8-1.el9.x86_64                   protobuf-3.14.0-16.el9.x86_64                     
  usbguard-1.0.0-16.el9.x86_64               usbguard-selinux-1.0.0-16.el9.noarch              

Complete!

```

`usbguard` is a service, we manage it as we would for any other services by using `systemctl`. To start the usbguard service, use `systemctl start`. `systemctl restart` after configuration changes. 
```
~]# systemctl start usbguard.service
~]# systemctl status usbguard
● usbguard.service - USBGuard daemon
   Loaded: loaded (/usr/lib/systemd/system/usbguard.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2017-06-06 13:29:31 CEST; 9s ago
     Docs: man:usbguard-daemon(8)
 Main PID: 4984 (usbguard-daemon)
   CGroup: /system.slice/usbguard.service
           └─4984 /usr/sbin/usbguard-daemon -k -c /etc/usbguard/usbguard-daem...
```

`systemctl enable` to ensure it is started on boot for persistance.  

## Write device policy rules with specific criteria to manage devices

Initial ruleset is empty:
```
[user@localhost ~]$ sudo ls -l /etc/usbguard/rules.conf 
-rw-------. 1 root root 0 Feb  7 17:02 /etc/usbguard/rules.conf
```

Generate initial local ruleset file by running `usbguard generate-policy > rules.conf`
Then install the generated ruleset file as per man page. Restart service after configuration changes.

```
[root@localhost /tmp]# usbguard generate-policy > rules.conf
[root@localhost /tmp]# install -m 0600 -o root -g root rules.conf /etc/usbguard/rules.conf
[root@localhost /tmp]# systemctl restart usbguard
```

To customize the USBGuard rule set, see`usbguard-rules.conf(5)` man page for more information. 
Selected output of `man 5 usbguard-rules.conf`:

```
INITIAL POLICY
       Using the usbguard CLI tool and its generate-policy subcommand, you can generate an initial policy for your system instead of writing one from scratch. The tool generates an allow policy for all devices connected to the system at the time of execution. It has several options to tweak the resulting policy, see usbguard(1) for further details.

       The policy will be printed out on the standard output. It’s a good idea to review the generated rules before using them on a system. The typical workflow for generating an initial policy could look like this:

               $ sudo usbguard generate-policy > rules.conf
               $ vi rules.conf
               (review/modify the rule set)
               $ sudo install -m 0600 -o root -g root rules.conf /etc/usbguard/rules.conf
               $ sudo systemctl restart usbguard
```

The `EXAMPLE POLICIES` section is also very useful during configuration.
```
EXAMPLE POLICIES
       The following examples show what to put into the rules.conf file in order to implement the given
       policy.

        1. Allow USB mass storage devices (USB flash disks) and block everything else

           This policy will block any device that isn’t just a mass storage device. Devices with a hidden keyboard interface in a USB flash disk will be blocked. Only devices with a single mass storage interface will be allowed to interact with the operating system. The policy consists of a single rule:

                   allow with-interface equals { 08:*:* }

           The blocking is implicit in this case because we didn’t write a block rule. Implicit blocking is useful to desktop users. A desktop applet listening to USBGuard events can ask the user for a decision if an implicit target was applied.
.
.
.
```

## Manage administrative policy and daemon configuration

The `usbguard-daemon.conf` is used to configure runtime parameters of the daemon.   

`usbguard-daemon.conf`:
```bash
AuditBackend=FileAudit
AuditFilePath=/var/log/usbguard/usbguard-audit.log
DeviceManagerBackend=uevent
DeviceRulesWithPort=false
ImplicitPolicyTarget=block
InsertedDevicePolicy=apply-policy
IPCAccessControlFiles=/etc/usbguard/IPCAccessControl.d/
IPCAllowedGroups=wheel
IPCAllowedUsers=root
PresentControllerPolicy=keep
PresentDevicePolicy=apply-policy
RestoreControllerDeviceState=false
RuleFile=/etc/usbguard/rules.conf
RuleFolder=/etc/usbguard/rules.d/
```

Example:
- To add users, configure `IPCAllowedUsers`.  These users will be allowed to use IPC interface to make changes to USBGuard. 
- To allow groups, configure `IPCAllowedGroups`
```
       IPCAllowedUsers=username [username ...]
           A space delimited list of usernames that the daemon will accept IPC connections from. Default:
           root

       IPCAllowedGroups=groupname [groupname ...]
           A space delimited list of groupnames that the daemon will accept IPC connections from.
```

`man usbguard-daemon.conf` for more information.

Example rules:
```
allow name "device-name"
allow hash "value"
```
Remember to add this to original rules file if it was not empty. Use `install` command to install the new rules file. Restart via `systemctl restart usbguard` to make new rules effective.

Details of USB ids can be found at `/usr/share/hwdata/usb.ids`
- { 08:*:* } typically Mass Storage Devices
- { 03:*:* } typically Human Interface Devices (HID)

# Reference
1. [Using USBGuard](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/security_guide/sec-using-usbguard)
2. [USBGuard Homepage](https://usbguard.github.io/)