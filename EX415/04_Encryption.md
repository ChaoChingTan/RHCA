# Objectives

1. [Encrypt and decrypt block devices using LUKS](#encrypt-and-decrypt-block-devices-using-luks)
2. [Configure encrypted storage persistence using NBDE](#configure-encrypted-storage-persistence-using-nbde)
3. Change encrypted storage passphrases.


## TLDR
1. CRYPTSETUP(8) - manage plain dm-crypt, LUKS, and other encrypted volumes
2. CRYPTTAB(5) - Configuration for encrypted block devices
3. FSTAB(5) - static information about the filesystems
4. TANG(8) - Network-Based Cryptographic Binding Server
5. CLEVIS(1) - Automated decryption policy framework
6. FIREWALL-CMD(1) - firewalld command line client
7. CRYPTSETUP-LUKSCHANGEKEY(8) - change an existing passphrase


## Encrypt and decrypt block devices using LUKS
Encrypt block devices using the `cryptsetup` command. The process secures the device using a passphrase, without which you will not be able to access the device. Output of `cryptsetup --help`:

```
cryptsetup 2.7.2 flags: UDEV BLKID KEYRING FIPS KERNEL_CAPI PWQUALITY 
Usage: cryptsetup [OPTION...] <action> <action-specific>

Help options:
  -?, --help                            Show this help message
      --usage                           Display brief usage
  -V, --version                         Print package version
      --active-name=STRING              Override device autodetection of dm device to be reencrypted
      --align-payload=SECTORS           Align payload at <n> sector boundaries - for luksFormat
      --allow-discards                  Allow discards (aka TRIM) requests for device
  -q, --batch-mode                      Do not ask for confirmation
      --cancel-deferred                 Cancel a previously set deferred device removal
  -c, --cipher=STRING                   The cipher used to encrypt the disk (see /proc/crypto)
      --debug                           Show debug messages
      --debug-json                      Show debug messages including JSON metadata
      --decrypt                         Decrypt LUKS2 device (remove encryption)
      --deferred                        Device removal is deferred until the last user closes it
      --device-size=bytes               Use only specified device size (ignore rest of device),DANGEROUS!
      --disable-blkid                   Disable blkid on-disk signature detection and wiping
      --disable-external-tokens         Disable loading of external LUKS2 token plugins
      --disable-keyring                 Disable loading volume keys via kernel keyring
      --disable-locks                   Disable locking of on-disk metadata
      --disable-veracrypt               Do not scan for VeraCrypt compatible device
      --dump-json-metadata              Dump info in JSON format (LUKS2 only)
      --dump-volume-key                 Dump volume key instead of keyslots info
      --encrypt                         Encrypt LUKS2 device (in-place encryption)
      --external-tokens-path=STRING     Path to directory with external token handlers (plugins).
      --force-password                  Disable password quality check (if enabled)
      --force-offline-reencrypt         Force offline LUKS2 reencryption and bypass active device detection
  -h, --hash=STRING                     The hash used to create the encryption key from the
                                        passphrase
      --header=STRING                   Device or file with separated LUKS header
      --header-backup-file=STRING       File with LUKS header and keyslots backup
      --hotzone-size=bytes              Maximal reencryption hotzone size
      --hw-opal                         Use HW OPAL encryption together with SW encryption
      --hw-opal-factory-reset           Wipe WHOLE OPAL disk on luksErase
      --hw-opal-only                    Use only HW OPAL encryption
      --init-only                       Initialize LUKS2 reencryption in metadata only
  -I, --integrity=STRING                Data integrity algorithm (LUKS2 only)
      --integrity-legacy-padding        Use inefficient legacy padding (old kernels)
      --integrity-no-journal            Disable journal for integrity device
      --integrity-no-wipe               Do not wipe device after format
  -i, --iter-time=msecs                 PBKDF iteration time for LUKS (in ms)
      --iv-large-sectors                Use IV counted in sector size (not in 512 bytes)
      --json-file=STRING                Read or write the json from or to a file
      --keep-key                        Do not change volume key
      --key-description=STRING          Key description
  -d, --key-file=STRING                 Read the key from a file
  -s, --key-size=BITS                   The size of the encryption key
  -S, --key-slot=INT                    Slot number for new key (default is first free)
      --keyfile-offset=bytes            Number of bytes to skip in keyfile
  -l, --keyfile-size=bytes              Limits the read from keyfile
      --keyslot-cipher=STRING           LUKS2 keyslot: The cipher used for keyslot encryption
      --keyslot-key-size=BITS           LUKS2 keyslot: The size of the encryption key
      --label=STRING                    Set label for the LUKS2 device
      --link-vk-to-keyring=STRING       Set keyring where to link volume key
      --luks2-keyslots-size=bytes       LUKS2 header keyslots area size
      --luks2-metadata-size=bytes       LUKS2 header metadata area size
      --new-keyfile=STRING              Read the key for a new slot from a file
      --new-keyfile-offset=bytes        Number of bytes to skip in newly added keyfile
      --new-keyfile-size=bytes          Limits the read from newly added keyfile
      --new-key-slot=INT                Slot number for new key (default is first free)
      --new-token-id=INT                Token number (default: any)
  -o, --offset=SECTORS                  The start offset in the backend device
      --pbkdf=STRING                    PBKDF algorithm (for LUKS2): argon2i, argon2id, pbkdf2
      --pbkdf-force-iterations=LONG     PBKDF iterations cost (forced, disables benchmark)
      --pbkdf-memory=kilobytes          PBKDF memory cost limit
      --pbkdf-parallel=threads          PBKDF parallel cost
      --perf-no_read_workqueue          Bypass dm-crypt workqueue and process read requests synchronously
      --perf-no_write_workqueue         Bypass dm-crypt workqueue and process write requests synchronously
      --perf-same_cpu_crypt             Use dm-crypt same_cpu_crypt performance compatibility option
      --perf-submit_from_crypt_cpus     Use dm-crypt submit_from_crypt_cpus performance compatibility option
      --persistent                      Set activation flags persistent for device
      --priority=STRING                 Keyslot priority: ignore, normal, prefer
      --progress-json                   Print progress data in json format (suitable for machine processing)
      --progress-frequency=secs         Progress line update (in seconds)
  -r, --readonly                        Create a readonly mapping
      --reduce-device-size=bytes        Reduce data device size (move data offset), DANGEROUS!
      --refresh                         Refresh (reactivate) device with new parameters
      --resilience=STRING               Reencryption hotzone resilience type
                                        (checksum,journal,none)
      --resilience-hash=STRING          Reencryption hotzone checksums hash
      --resume-only                     Resume initialized LUKS2 reencryption only
      --sector-size=INT                 Encryption sector size (default: 512 bytes)
      --serialize-memory-hard-pbkdf     Use global lock to serialize memory hard PBKDF (OOM workaround)
      --shared                          Share device with another non-overlapping crypt segment
  -b, --size=SECTORS                    The size of the device
  -p, --skip=SECTORS                    How many sectors of the encrypted data to skip at the beginning
      --subsystem=STRING                Set subsystem label for the LUKS2 device
      --test-args                       Do not run action, just validate all command line parameters
      --test-passphrase                 Do not activate device, just check passphrase
  -t, --timeout=secs                    Timeout for interactive passphrase prompt (in seconds)
      --token-id=INT                    Token number (default: any)
      --token-only                      Do not ask for passphrase if activation by token fails
      --token-replace                   Replace the current token
      --token-type=STRING               Restrict allowed token types used to retrieve LUKS2 key
      --tcrypt-backup                   Use backup (secondary) TCRYPT header
      --tcrypt-hidden                   Use hidden header (hidden TCRYPT device)
      --tcrypt-system                   Device is system TCRYPT drive (with bootloader)
  -T, --tries=INT                       How often the input of the passphrase can be retried
  -M, --type=STRING                     Type of device metadata: luks, luks1, luks2, plain, loopaes, tcrypt,bitlk
      --unbound                         Create or dump unbound LUKS2 keyslot (unassigned to data segment) or LUKS2 token (unassigned to keyslot)
      --use-random                      Use /dev/random for generating volume key
      --use-urandom                     Use /dev/urandom for generating volume key
      --uuid=STRING                     UUID for device to use
      --veracrypt                       Scan also for VeraCrypt compatible device
      --veracrypt-pim=INT               Personal Iteration Multiplier for VeraCrypt compatible device
      --veracrypt-query-pim             Query Personal Iteration Multiplier for VeraCrypt compatible device
  -v, --verbose                         Shows more detailed error messages
  -y, --verify-passphrase               Verifies the passphrase by asking for it twice
      --volume-key-file=STRING          Use the volume key from file
      --volume-key-keyring=STRING       Use the specified keyring key as a volume key
  -B, --block-size=MiB                  Reencryption block size
  -N, --new                             Create new header on not encrypted device
      --use-directio                    Use direct-io when accessing devices
      --use-fsync                       Use fsync after each block
      --write-log                       Update log file after every block
      --dump-master-key                 Alias for --dump-volume-key
      --master-key-file=STRING          Alias for --dump-volume-key-file

<action> is one of:
	open <device> [--type <type>] [<name>] - open device as <name>
	close <name> - close device (remove mapping)
	resize <name> - resize active device
	status <name> - show device status
	benchmark [--cipher <cipher>] - benchmark cipher
	repair <device> - try to repair on-disk metadata
	reencrypt <device> - reencrypt LUKS2 device
	erase <device> - erase all keyslots (remove encryption key)
	convert <device> - convert LUKS from/to LUKS2 format
	config <device> - set permanent configuration options for LUKS2
	luksFormat <device> [<new key file>] - formats a LUKS device
	luksAddKey <device> [<new key file>] - add key to LUKS device
	luksRemoveKey <device> [<key file>] - removes supplied key or key file from LUKS device
	luksChangeKey <device> [<key file>] - changes supplied key or key file of LUKS device
	luksConvertKey <device> [<key file>] - converts a key to new pbkdf parameters
	luksKillSlot <device> <key slot> - wipes key with number <key slot> from LUKS device
	luksUUID <device> - print UUID of LUKS device
	isLuks <device> - tests <device> for LUKS partition header
	luksDump <device> - dump LUKS partition information
	tcryptDump <device> - dump TCRYPT device information
	bitlkDump <device> - dump BITLK device information
	fvault2Dump <device> - dump FVAULT2 device information
	luksSuspend <device> - Suspend LUKS device and wipe key (all IOs are frozen)
	luksResume <device> - Resume suspended LUKS device
	luksHeaderBackup <device> - Backup LUKS device header and keyslots
	luksHeaderRestore <device> - Restore LUKS device header and keyslots
	token <add|remove|import|export> <device> - Manipulate LUKS2 tokens

You can also use old <action> syntax aliases:
	open: create (plainOpen), luksOpen, loopaesOpen, tcryptOpen, bitlkOpen, fvault2Open
	close: remove (plainClose), luksClose, loopaesClose, tcryptClose, bitlkClose, fvault2Close

<name> is the device to create under /dev/mapper
<device> is the encrypted device
<key slot> is the LUKS key slot number to modify
<key file> optional key file for the new key for luksAddKey action

Default compiled-in metadata format is LUKS2 (for luksFormat action).

LUKS2 external token plugin support is enabled.
LUKS2 external token plugin path: /usr/lib64/cryptsetup.

Default compiled-in key and passphrase parameters:
	Maximum keyfile size: 8192kB, Maximum interactive passphrase length 512 (characters)
Default PBKDF for LUKS1: pbkdf2, iteration time: 2000 (ms)
Default PBKDF for LUKS2: argon2id
	Iteration time: 2000, Memory required: 1048576kB, Parallel threads: 4

Default compiled-in device cipher parameters:
	loop-AES: aes, Key 256 bits
	plain: aes-cbc-essiv:sha256, Key: 256 bits, Password hashing: ripemd160
	LUKS: aes-xts-plain64, Key: 256 bits, LUKS header hashing: sha256, RNG: /dev/urandom
	LUKS: Default keysize with XTS mode (two internal keys) will be doubled.
```

### General flow 
1. Format the device by using `cryptsetup luksFormat`. This is the step where you input the passphrase for the device. 
2. `open` device as name, for example `/dev/mapper/encrypted`.  You will require the passphrase which you have setup in the step above. If this step is successful, device can be seen under `/dev/mapper`.
3. Make the new file system on this device. For the example above, this will be `/dev/mapper/encrypted`
4. Mount the device as per normal, eg: `mount /dev/mapper/encrypted /mnt/`
5. After you are done with the encrypted volume, `umount` the mount point
6. Finally use `cryptsetup close <name>` - close device (remove mapping)


## Configure encrypted storage persistence using NBDE
To ensure encryption storage persistence **without** using NBDE, you will need to modify the following files:
- /etc/crypttab - Before modifying this file, run the command `cryptsetup luksAddKey <device> [<new key file>]` to add the keyfile. This file can be generated via a command like `dd if=/dev/urandom of=[<new key file>] bs=4096 count=1`. Permissions of the key file should be 0600.

```
DESCRIPTION
       The /etc/crypttab file describes encrypted block devices that are set up during
       system boot.

       Empty lines and lines starting with the "#" character are ignored. Each of the
       remaining lines describes one encrypted block device. Fields are delimited by white
       space.

       Each line is in the form

           volume-name encrypted-device key-file options
    .
    .
    .
```
- /etc/fstab - put the `/dev/mapper` device in fstab.
```
/dev/mapper/<device>    /mnt    xfs    defaults    0   0 
```

For encrypted storage persistence using NBDE, you will need a client-server setup using `tang` on the server side and `clevis` on the client.  `tang` is a Network-Based Cryptographic Binding Server. After installation, start and enable the service, according to instruction from man pages:
```
sudo systemctl enable tangd.socket --now
```
Database directory is initially empty at `/var/db/tang/`.  Get key advertisement by running the following command on the tang server:
```bash
curl http://$(hostname)/adv
```
You will see the server key advertisement after which examining the database directory will show 2 `.jwk` files created - an encryption key and a signing key.

As for the client, ensure that the package `clevis` is installed. Test out connectivity to the server by running a curl before continuing with the rest of the configuration:
```bash
curl http://[TANG-SERVERNAME]/adv
```

Note that you may have to open up ports using the `firewall-cmd` on the tang server.  Consult the manual page on `firewall-cmd` details.  

Ensure that the packages `clevis` and `clevis-dracut` are installed on the client system. 

`clevis` luks binding, refer to the manual page:
```
LUKS BINDING
       Clevis can be used to bind an existing LUKS volume to its automation policy. This is accomplished with
       a simple command:

           $ clevis luks bind -d /dev/sda tang '{"url":...}'

       This command performs four steps:

        1. Creates a new key with the same entropy as the LUKS master key — maximum entropy bits is 256.

        2. Encrypts the new key with Clevis.

        3. Stores the Clevis JWE in the LUKS header.

        4. Enables the new key for use with LUKS.

       This disk can now be unlocked with your existing password as well as with the Clevis policy. Clevis
       provides two unlockers for LUKS volumes. First, we provide integration with Dracut to automatically
       unlock your root volume during early boot. Second, we provide integration with UDisks2 to
       automatically unlock your removable media in your desktop session.
```

Example output:
```
[user@localhost ~]$ sudo clevis luks bind -d /dev/vdb1 tang '{"url": "http://foo.bar.baz.qux"}'
[sudo] password for admin: 
The advertisement contains the following signing keys:

lVqV16K4dV3flOA6APajnKpY4sU

Do you wish to trust these keys? [ynYN] Y
Enter existing LUKS password: 
No key available with this passphrase.
[user@localhost ~]$ sudo clevis luks bind -d /dev/vdb1 tang '{"url": "http://foo.bar.baz.qux"}'
The advertisement contains the following signing keys:

lVqV16K4dV3flOA6APajnKpY4sU

Do you wish to trust these keys? [ynYN] Y
Enter existing LUKS password: 

```

Verify after running this command by using `cryptsetup luksDump`.  You should see an additional key slot used, generated by the tang/clevis system.

Regenerate initial ramdisk. This step is required else the network may not come up after reboot.
```bash
dracut -fv --regenerate-all --hostonly-cmdline --add "network clevis"
```


## Change encrypted storage passphrases
Use `cryptsetup luksChangeKey` command:
```
SYNOPSIS
       cryptsetup luksChangeKey [<options>] <device> [<new key file>]
```


# References
1. Tweedale, F. (2020, January 15). Clevis and Tang: securing your secrets at rest [Video]. YouTube.(https://www.youtube.com/watch?v=Dk6ZuydQt9I)
2. Red Hat, Inc. (2025). [Configuring automated unlocking of encrypted volumes by using policy-based decryption.](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/security_hardening/configuring-automated-unlocking-of-encrypted-volumes-using-policy-based-decryption_security-hardening) In Security hardening (Chapter 10). Red Hat. Retrieved September 9, 2025