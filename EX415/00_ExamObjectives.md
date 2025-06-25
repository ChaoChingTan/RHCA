# EX415 Exam Objectives

## [Use Red Hat Ansible® Engine](./01_Red_Hat_Ansible.md)

    Install Red Hat Ansible Engine on a control node.
    Configure managed nodes.
    Configure simple inventories.
    Perform basic management of systems.
    Run a provided playbook against specified nodes.


## [Implement access controls for automation controller](./02_Automation_Controller.md)

    Create and restrict an inventory to an automation controller user
    Restrict a credential and/or a project to an automation controller user
    Be able to create and launch a template as an automation controller user


## Configure intrusion detection

- Install [AIDE](#install-aide)
- Configure AIDE to monitor critical system files.

### Install AIDE

Refer to the Redhat [Installation Guide](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/checking-integrity-with-aide_security-hardening#installing-aide_checking-integrity-with-aide) 

Install AIDE package

```bash
sudo dnf install aide
```

Generate an initial database:

```bash
sudo aide --init

Start timestamp: 2025-03-24 16:04:26 +0800 (AIDE 0.16)
AIDE initialized database at /var/lib/aide/aide.db.new.gz

Number of entries:	272293

---------------------------------------------------
The attributes of the (uncompressed) database(s):
---------------------------------------------------

/var/lib/aide/aide.db.new.gz
  MD5      : ZQCo3XQVs/xW9RyuuSjMtA==
  SHA1     : pynN0S2upBj3gLWtAUwl6yaEkpk=
  RMD160   : lG9+kAEA1ttKWzwZiLgcF9ee3qY=
  TIGER    : 3TfxpeoGHpt6JSj/zvbLs1Y+Ou6W3Px9
  SHA256   : e+s58DLXi5/B3kDNYLE7ls44qtREvmja
             LHx9xj5Qahg=
  SHA512   : DGL6VDF9wcC+NDHCOS2e+0pnyYQW7TeK
             lOE/R5SaTZWAfFtmcC3UEM49bgIKC5Sz
             LbUhXkHM1e4AaT5zy/gRaA==


End timestamp: 2025-03-24 16:05:16 +0800 (run time: 0m 50s)
```

### Configure AIDE to monitor critical system files

`aide --init` command checks just a set of directories and files defined in the /etc/aide.conf file. 

To include additional directories or files in the AIDE database, and to change their watched parameters, edit `/etc/aide.conf` accordingly.


## Configure encrypted storage
    Encrypt and decrypt block devices using LUKS.
    Configure encrypted storage persistence using NBDE.
    Change encrypted storage passphrases.


## [Restrict USB devices](./05_USBGuard.md)

    Install USBGuard.
    Write device policy rules with specific criteria to manage devices.
    Manage administrative policy and daemon configuration.


## Manage system login security using pluggable authentication modules (PAMs)

    Configure password quality requirements.
    Configure failed login policy.
    Modify PAM configuration files and parameters.


## Configure system auditing

    Write rules to log auditable events.
    Enable prepackaged rules.
    Produce audit reports.


## Configure SELinux

    Enable SELinux on a host running a simple application.
    Interpret SELinux violations and determine remedial action.
    Restrict user activity with SELinux user mappings.
    Analyze and correct existing SELinux configurations.


## Enforce security compliance

    Install OpenSCAP and OpenSCAP Workbench.
    Scan hosts for security compliance
    Tailor security policy
    Scan individual hosts for security compliance
    Generate and apply a playbook from customized XML for remediation of inventory hosts


As with all Red Hat performance-based exams, configurations must persist after reboot without intervention.

# Reference
- [EX415](https://www.redhat.com/en/services/training/ex415-red-hat-certified-specialist-security-linux-exam) Red Hat Certified Specialist in Security: Linux exam
- [AIDE](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/checking-integrity-with-aide_security-hardening) Checking integrity with AIDE