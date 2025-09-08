# Objectives

Manage system login security using pluggable authentication modules (PAMs)
1. [Configure password quality requirements](#configure-password-quality-requirements)
2. [Configure failed login policy](#configure-failed-login-policy)
3. Modify PAM configuration files and parameters

## TLDR
1. PAM(8)
2. PAM.CONF(5)
3. `man -k pam_` - Quickly find PAM modules on system
4. PAM_PWQUALITY(8)
5. PAM_PWHISTORY(8)
6. PAM_UNIX(8)
7. PAM_DENY(8)
8. PAM_PERMIT(8)
9. PAM_FAILDELAY(8)
10. PAM_FAILLOCK(8)
11. PAM_TIME(8)
12. PAM_SECURETTY(8)
13. PAM_LIMITS(8)
14. PAM_TTY_AUDIT(8)

## PAM Overview

`man pam`:
```
DESCRIPTION
       This manual is intended to offer a quick introduction to Linux-PAM. For more information the reader is directed to the Linux-PAM system administrators' guide.
.
.
.
       Linux-PAM separates the tasks of authentication into four independent management groups: account management; authentication management; password management; and session management.
.
.
.
       account - provide account verification types of service: has the user's password expired?; is this user permitted access to the requested service?

       authentication - authenticate a user and set up user credentials. Typically this is via some challenge-response request that the user must satisfy: if you are who you claim to be please enter your password. 
.
.
.
       password - this group's responsibility is the task of updating authentication mechanisms. Typically, such services are strongly coupled to those of the auth group. Some authentication mechanisms lend themselves well to being updated with such a function. Standard UN*X password-based access is the
       obvious example: please enter a replacement password.

       session - this group of tasks cover things that should be done prior to a service being given and after it is withdrawn. Such tasks include the maintenance of audit trails and the mounting of the user's home directory. The session management group is important as it provides both an opening and
       closing hook for modules to affect the services available to a user.
```

`man pam.conf`
```
DESCRIPTION
       When a PAM aware privilege granting application is started, it activates its attachment to the PAM-API. This activation performs a number of tasks, the most important being the reading of the configuration file(s): /etc/pam.conf. Alternatively, this may be the contents of the /etc/pam.d/ directory. The presence of this directory will cause Linux-PAM to ignore /etc/pam.conf.

       These files list the PAMs that will do the authentication tasks required by this service, and the appropriate behavior of the PAM-API in the event that individual PAMs fail.

       The syntax of the /etc/pam.conf configuration file is as follows. The file is made up of a list of rules, each rule is typically placed on a single line, but may be extended with an escaped end of line: `\<LF>'. Comments are preceded with `#' marks and extend to the next end of line.

       The format of each rule is a space separated collection of tokens, the first three being case-insensitive:

        service type control module-path module-arguments
       
       The syntax of files contained in the /etc/pam.d/ directory, are identical except for the absence of any service field. In this case, the service is the name of the file in the /etc/pam.d/ directory. This filename must be in lower case.
       .
       .
       .
       The third field, control, indicates the behavior of the PAM-API should the module fail to succeed in its authentication task. There are two types of syntax for this control field: the simple one has a single simple keyword; the more complicated one involves a square-bracketed selection of value=action
       pairs.

       For the simple (historical) syntax valid control values are:

       required
           failure of such a PAM will ultimately lead to the PAM-API returning failure but only after the remaining stacked modules (for this service and type) have been invoked.

       requisite
           like required, however, in the case that such a module returns a failure, control is directly returned to the application or to the superior PAM stack. The return value is that associated with the first required or requisite module to fail. Note, this flag can be used to protect against the possibility of a user getting the opportunity to enter a password over an unsafe medium. It is conceivable that such behavior might inform an attacker of valid accounts on a system. This possibility should be weighed against the not insignificant concerns of exposing a sensitive
           password in a hostile environment.

       sufficient
           if such a module succeeds and no prior required module has failed the PAM framework returns success to the application or to the superior PAM stack immediately without calling any further modules in the stack. A failure of a sufficient module is ignored and processing of the PAM module
           stack continues unaffected.
       .
       .
       .
```

## Configure password quality requirements
Related PAM modules:
- `pam_pwquality`, see EXAMPLES in man page:
```
.
.
.
        #
        # These lines require the user to select a password with a minimum
        # length of 8 and with at least 1 digit number, 1 upper case letter,
        # and 1 other character
        #
        password required pam_pwquality.so \
                      dcredit=-1 ucredit=-1 ocredit=-1 lcredit=0 minlen=8
        password required pam_unix.so use_authtok nullok sha256
.
.
.
```
- `pam_pwhistory` and config file at `/etc/security/pwhistory.conf`
- `pam_unix` - Module for traditional password authentication
```
EXAMPLES
       An example usage for /etc/pam.d/login would be:

           # Authenticate the user
           auth       required   pam_unix.so
           # Ensure users account and password are still active
           account    required   pam_unix.so
           # Change the user's password, but at first check the strength
           # with pam_passwdqc(8)
           password   required   pam_passwdqc.so config=/etc/passwdqc.conf
           password   required   pam_unix.so use_authtok nullok yescrypt
           session    required   pam_unix.so
```
- `pam_deny`
```
EXAMPLES
           #%PAM-1.0
           #
           # If we don't have config entries for a service, the
           # OTHER entries are used. To be secure, warn and deny
           # access to everything.
           other auth     required       pam_warn.so
           other auth     required       pam_deny.so
           other account  required       pam_warn.so
           other account  required       pam_deny.so
           other password required       pam_warn.so
           other password required       pam_deny.so
           other session  required       pam_warn.so
           other session  required       pam_deny.so
```
- `pam_permit` - Dangerous for obvious reasons, use with **extreme caution**


## Configure failed login policy
Related PAM modules:
- `pam_faildelay`
```
EXAMPLES
       The following example will set the delay on failure to 10 seconds:

           auth  optional  pam_faildelay.so  delay=10000000
```

- `pam_faillock` and configration file `/etc/security/faillock.conf`
```
       /etc/pam.d/config file example:

           auth     required       pam_securetty.so
           auth     required       pam_env.so
           auth     required       pam_nologin.so
           # optionally call: auth requisite pam_faillock.so preauth
           # to display the message about account being locked
           auth     [success=1 default=bad] pam_unix.so
           auth     [default=die]  pam_faillock.so authfail
           auth     sufficient     pam_faillock.so authsucc
           auth     required       pam_deny.so
           account  required       pam_unix.so
           password required       pam_unix.so shadow
           session  required       pam_selinux.so close
           session  required       pam_loginuid.so
           session  required       pam_unix.so
           session  required       pam_selinux.so open
```


## Modify PAM configuration files and parameters
Other relevant modules listed below. Configuration files are typically found in `/etc/security` directory.
- `pam_time` - PAM module for time control access
- `pam_securetty` - Limit root login to special devices
- `pam_limits` - PAM module to limit resources
- `pam_tty_audit` - Enable or disable TTY auditing for specified users