# Objectives

Configure intrusion detection
1. [Install AIDE](#install-aide)
2. [Configure AIDE to monitor critical system files](#configure-aide-to-monitor-critical-system-files)

## TLDR
1. aide(1)
2. aide.conf(5)

## Install AIDE

Installation via `yum` or `dnf`:
```bash
yum install aide -y
```

Refer to manpage: 
```bash
man aide
```

`FILES` section under manpage:
```
.
.
.
FILES
       /etc/aide.conf
              Default aide configuration file.

       /var/lib/aide/aide.db
              Default aide database.

       /var/lib/aide/aide.db.new
              Default aide output database.

```

Typical AIDE workflow:
- sudo aide --init
- sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz 
- sudo aide --check
- sudo aide --update
- sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz 

```
[user@localhost ~]$ aide --help
Aide 0.16 

Usage: aide [options] command

Commands:
  -i, --init		Initialize the database
  -C, --check		Check the database
  -u, --update		Check and update the database non-interactively
  -E, --compare		Compare two databases

Miscellaneous:
  -D, --config-check	Test the configuration file
  -v, --version		Show version of AIDE and compilation options
  -h, --help		Show this help message

Options:
  -c [cfgfile]	--config=[cfgfile]	Get config options from [cfgfile]
  -l [REGEX]	--limit=[REGEX]		Limit command to entries matching [REGEX]
  -B "OPTION"	--before="OPTION"	Before configuration file is read define OPTION
  -A "OPTION"	--after="OPTION"	After configuration file is read define OPTION
  -r [reporter]	--report=[reporter]	Write report output to [reporter] url
  -V[level]	--verbose=[level]	Set debug message level to [level]
```


## Configure AIDE to monitor critical system files

Configuration file is located at `/etc/aide.conf`. To configure sending to syslog or email:
```
report_url=file:@@{LOGDIR}/aide.log
report_url=stdout
#report_url=stderr
#NOT IMPLEMENTED report_url=mailto:root@foo.com
#NOT IMPLEMENTED report_url=syslog:LOG_AUTH
```

Rules:
```
# These are the default rules.
#
#p:      permissions
#i:      inode:
#n:      number of links
#u:      user
#g:      group
#s:      size
#b:      block count
#m:      mtime
#a:      atime
#c:      ctime
#S:      check for growing size
#acl:           Access Control Lists
#selinux        SELinux security context
#xattrs:        Extended file attributes
#md5:    md5 checksum
#sha1:   sha1 checksum
#sha256:        sha256 checksum
#sha512:        sha512 checksum
#rmd160: rmd160 checksum
#tiger:  tiger checksum

#haval:  haval checksum (MHASH only)
#gost:   gost checksum (MHASH only)
#crc32:  crc32 checksum (MHASH only)
#whirlpool:     whirlpool checksum (MHASH only)

#R:             p+i+n+u+g+s+m+c+acl+selinux+xattrs+md5
#L:             p+i+n+u+g+acl+selinux+xattrs
#E:             Empty group
#>:             Growing logfile p+u+g+i+n+S+acl+selinux+xattrs

# You can create custom rules like this.
# With MHASH...
# ALLXTRAHASHES = sha1+rmd160+sha256+sha512+whirlpool+tiger+haval+gost+crc32
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+tiger
# Everything but access time (Ie. all changes)
EVERYTHING = R+ALLXTRAHASHES
```
Configuration:
```
# Next decide what directories/files you want in the database.

/boot       CONTENT_EX
/opt        CONTENT
```
