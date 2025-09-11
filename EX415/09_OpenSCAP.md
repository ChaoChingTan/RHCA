# Objectives

1. [Install OpenSCAP and OpenSCAP Workbench](#install-openscap-and-openscap-workbench)
2. [Scan hosts for security compliance](#scan-hosts-for-security-compliance)
3. [Tailor security policy](#tailor-security-policy)
4. [Scan individual hosts for security compliance](#scan-individual-hosts-for-security-compliance)
5. [Generate and apply a playbook from customized XML for remediation of inventory hosts](#generate-and-apply-a-playbook-from-customized-xml-for-remediation-of-inventory-hosts)


## Install OpenSCAP and OpenSCAP Workbench
Install openscap:

```bash
$ sudo dnf install openscap-scanner
```

Install openscap workbench:
```bash
$ sudo dnf install scap-workbench
```

Installing scap-security-guide, gives definitions of security standards
```bash
$ sudo dnf install scap-security-guide
```

## Scan hosts for security compliance

### Scanning using a graphical interface
The most straightforward way to scan is use the GUI version by running the command `scap-workbench`.

The online documentation can be found at https://www.open-scap.org/getting-started/ where you will be able to find screenshots for the output of running the workbench in GUI mode.  

### openscap-workbench
Launch the openscap workbench `scap-workbench`.  At this point you can select the profile and target host, then choose **scan**.  

### Scanning via command line
This is done using the `oscap` tool.  This tool is installed via the package `oscap-scanner`.  

The oscap scanner has no policies.  On RHEL, default policies are provided by SCAP Security Guide (SSG). This was installed via the `scap-security-guide` package.  The policy files are installed to location `/usr/share/xml/scap/ssg/content/`.  Profiles can be listed with the `oscap info` command.  

To evaluate against a policy, use the `oscap xccdf eval` subcommands.  To find out more about these subcommands, use the `--help` option.  

```
$ oscap xccdf eval --help
oscap -> xccdf -> eval

Perform evaluation driven by XCCDF file and use OVAL as checking engine

Usage: oscap [options] xccdf eval [options] INPUT_FILE [oval-definitions-files]

Common options:
   --verbose <verbosity_level>   - Turn on verbose mode at specified verbosity level.
                                   Verbosity level must be one of: DEVEL, INFO, WARNING, ERROR.
   --verbose-log-file <file>     - Write verbose information into file.

INPUT_FILE - XCCDF file or a source data stream file

Options:
   --profile <name>              - The name of Profile to be evaluated.
   --rule <name>                 - The name of a single rule to be evaluated.
   --skip-rule <name>            - The name of the rule to be skipped.
   --reference <NAME:ID>         - Evaluate only rules that have the given reference.
   --tailoring-file <file>       - Use given XCCDF Tailoring file.
   --tailoring-id <component-id> - Use given DS component as XCCDF Tailoring file.
   --cpe <name>                  - Use given CPE dictionary or language (autodetected)
                                   for applicability checks.
   --oval-results                - Save OVAL results as well.
   --check-engine-results        - Save results from check engines loaded from plugins as well.
   --export-variables            - Export OVAL external variables provided by XCCDF.
   --results <file>              - Write XCCDF Results into file.
   --results-arf <file>          - Write ARF (result data stream) into file.
   --stig-viewer <file>          - Writes XCCDF results into FILE in a format readable by DISA STIG Viewer
   --thin-results                - Thin Results provides only minimal amount of information in OVAL/ARF results.
                                   The option --without-syschar is automatically enabled when you use Thin Results.
   --without-syschar             - Don't provide system characteristic in OVAL/ARF result files.
   --report <file>               - Write HTML report into file.
   --skip-valid                  - Skip validation.
   --skip-validation
   --skip-signature-validation   - Skip data stream signature validation.
                                   (only applicable for source data streams)
   --enforce-signature           - Process only signed data streams.
   --fetch-remote-resources      - Download remote content referenced by XCCDF.
   --local-files <dir>           - Use locally downloaded copies of remote resources stored in the given directory.
   --progress                    - Switch to sparse output suitable for progress reporting.
                                   Format is "$rule_id:$result\n".
   --progress-full               - Switch to sparse but a bit more saturated output also suitable for progress reporting.
                                   Format is "$rule_id|$rule_title|$result\n".
   --datastream-id <id>          - ID of the data stream in the collection to use.
                                   (only applicable for source data streams)
   --xccdf-id <id>               - ID of component-ref with XCCDF in the data stream that should be evaluated.
                                   (only applicable for source data streams)
   --benchmark-id <id>           - ID of XCCDF Benchmark in some component in the data stream that should be evaluated.
                                   (only applicable for source data streams)
                                   (only applicable when datastream-id AND xccdf-id are not specified)
   --remediate                   - Automatically execute XCCDF fix elements for failed rules.
                                   Use of this option is always at your own risk.
```

## Tailor security policy
Choose "Customize" in the `oscap-workbench` GUI.  Essentially you start off with a base policy and add / remove items as required.  

## Scan individual hosts for security compliance
The openscap workbench can be configured to scan remote hosts instead of local host.  You need to provide the username and credentials to access the remote machine.  The remote machine will also need to have `openscap-scanner` package installed.

Ensure that sudoer is setup to not require password or scanning will fail.  Note that if the OS versions are different, you will need to install the `scap-security-guide` on the target host and copy the associated ssg file under `/usr/share/xml/scap/ssg/content/` over.  

Once the setup is done, remote scanning works pretty much the same way as local scanning.  


## Generate and apply a playbook from customized XML for remediation of inventory hosts
You can save a remediation ansible playbook post scanning from the openscap workbench.  This is done by choosing "Generate remediation role" and selecting "ansible".  This saves the ansible playbook which can be used to remediate the non compliance on the machine.

Prior to this you will have to setup ansible access from the scanning host if scanning and remediation were to be be done from a remote host.  Refer to the chapter on ansible for details on how to set this up.