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


## [Configure intrusion detection](./03_Intrusion_Detection.md)

- Install AIDE 
- Configure AIDE to monitor critical system files.


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