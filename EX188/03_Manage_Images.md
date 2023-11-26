# Manage Images

## Objectives

- Understand private registry security
  - [Image Registries](#image-registries)
  - podman login
- Interact with many different registries
  - [skopeo command](#skopeo-command)
- Understand and use image tags
- Push and pull images from and to registries
  - podman pull
  - podman push
- Back up an image with its layers and meta data 
  - [podman save](#podman-save)
  - [podman load](#podman-load)
- vs. backup a container state
  - podman commit
  - podman save
  - podman load


## Image Registries
podman searches for images in registries defined in `/etc/containers/registries.conf`.

For unqualified searches, the registries are searched in the order specified by the `unqualified-search-registries` key in the configuration file:

```console
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io"]
```

### registry.access.redhat.com vs registry.redhat.io
- `registry.access.redhat.com` - no login required
- `registry.redhat.io` - login required


## `skopeo` command

`skopeo` can be used to inspect images in remote registries or copy images between registries

```bash
man skopeo
```

Common subcommands
```quote
inspect
  - skopeo inspect docker://registry.access.redhat.com/ubi8
copy
  - skopeo copy docker://registry.access.redhat.com/ubi8 dir:ubi8
```


## podman save

The `podman save` command save image(s) to an archive.  Syntax:

```console
podman save [options] name[:tag]
```

First check the images on the system with command:

```bash
podman images
```

```console
[user@localhost ~]$ podman images
REPOSITORY                       TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8  latest      86b358a425da  3 weeks ago  213 MB
[user@localhost ~]$ 
```
In our example above, there is an `ubi8:latest` image on our system.

To backup this image, run the command `podman save`:

```bash
podman save ubi8:latest -o ubi8.tar
```

Output:

```console
[user@localhost ~]$ podman save ubi8:latest -o ubi8.tar
Copying blob d93817448019 done  
Copying config 86b358a425 done  
Writing manifest to image destination
[user@localhost ~]$ 
```

The output archive file is `ubi8.tar` in the example above.  


## podman load

The `podman load` command load image(s) from a tar archive into container storage.  Syntax:

```console
podman load [options]
```

Example usage, continuing from the example from [podman save](#podman-save), let's first remove the ubi8 image from the system by running `podman rmi`:

```bash
podman rmi -a
```

Output:

```console
[user@localhost ~]$ podman rmi -a
Untagged: registry.access.redhat.com/ubi8:latest
Deleted: 86b358a425dac33c8dc6a96226af378f94c2a949dbcbb01c66f094a6d1fd036b
[user@localhost ~]$ 
```

There are no longer any images on this system, as shown by the output of the `podman images` command:

```console
[user@localhost ~]$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
[user@localhost ~]$ 
```

To load the `ubi8:latest` image from the tar archive, run the `podman load` command:

```bash
podman load -i ubi8.tar
```

Output:

```console
[user@localhost ~]$ podman load -i ubi8.tar 
Getting image source signatures
Copying blob d93817448019 done  
Copying config 86b358a425 done  
Writing manifest to image destination
Loaded image: registry.access.redhat.com/ubi8:latest
[user@localhost ~]$ 
```
Verify that the image is now found in the container storage by running the `podman images` command:

```console
[user@localhost ~]$ podman images
REPOSITORY                       TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8  latest      86b358a425da  3 weeks ago  213 MB
[user@localhost ~]$ 
```


## podman commit

`podman commit` create a new image based on the changed container.  Syntax:

```console
podman commit [options] container [image]
```

Consider the following host which has a container `nmap-test` currently running:

```console
[user@localhost ~]$ podman ps -a
CONTAINER ID  IMAGE                                   COMMAND     CREATED        STATUS        PORTS       NAMES
da2a7a3b1281  registry.access.redhat.com/ubi8:latest  bash        6 seconds ago  Up 7 seconds              nmap-test
[user@localhost ~]$ 
```

Current container does not have nmap installed:

```console
[user@localhost ~]$ podman attach nmap-test 
[root@da2a7a3b1281 /]# nmap
bash: nmap: command not found
[root@da2a7a3b1281 /]# 
```

Let's proceed to install `nmap` in the container:

```console
[root@da2a7a3b1281 /]# dnf install -y nmap
Updating Subscription Management repositories.
Unable to read consumer identity
subscription-manager is operating in container mode.
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)                                                5.9 MB/s |  58 MB     00:09    
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)                                                   3.8 MB/s |  63 MB     00:16    
Red Hat Universal Base Image 8 (RPMs) - BaseOS                                                          150 kB/s | 717 kB     00:04    
Red Hat Universal Base Image 8 (RPMs) - AppStream                                                       546 kB/s | 3.0 MB     00:05    
Red Hat Universal Base Image 8 (RPMs) - CodeReady Builder                                                22 kB/s | 102 kB     00:04    
Last metadata expiration check: 0:00:01 ago on Sun Nov 26 09:15:31 2023.
Dependencies resolved.
========================================================================================================================================
 Package                   Architecture          Version                          Repository                                       Size
========================================================================================================================================
Installing:
 nmap                      x86_64                2:7.92-1.el8                     rhel-8-for-x86_64-appstream-rpms                5.9 M
Installing dependencies:
 libibverbs                x86_64                46.0-1.el8.1                     rhel-8-for-x86_64-baseos-rpms                   400 k
 libpcap                   x86_64                14:1.9.1-5.el8                   rhel-8-for-x86_64-baseos-rpms                   169 k
 nmap-ncat                 x86_64                2:7.92-1.el8                     rhel-8-for-x86_64-appstream-rpms                243 k

Transaction Summary
========================================================================================================================================
Install  4 Packages

Total download size: 6.7 M
Installed size: 26 M
Downloading Packages:
(1/4): libpcap-1.9.1-5.el8.x86_64.rpm                                                                    66 kB/s | 169 kB     00:02    
(2/4): nmap-ncat-7.92-1.el8.x86_64.rpm                                                                   94 kB/s | 243 kB     00:02    
(3/4): nmap-7.92-1.el8.x86_64.rpm                                                                       1.8 MB/s | 5.9 MB     00:03    
(4/4): libibverbs-46.0-1.el8.1.x86_64.rpm                                                               171 kB/s | 400 kB     00:02    
----------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                   1.4 MB/s | 6.7 MB     00:04     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                1/1 
  Installing       : libibverbs-46.0-1.el8.1.x86_64                                                                                 1/4 
  Installing       : libpcap-14:1.9.1-5.el8.x86_64                                                                                  2/4 
  Installing       : nmap-ncat-2:7.92-1.el8.x86_64                                                                                  3/4 
  Running scriptlet: nmap-ncat-2:7.92-1.el8.x86_64                                                                                  3/4 
  Installing       : nmap-2:7.92-1.el8.x86_64                                                                                       4/4 
  Running scriptlet: nmap-2:7.92-1.el8.x86_64                                                                                       4/4 
  Verifying        : nmap-2:7.92-1.el8.x86_64                                                                                       1/4 
  Verifying        : nmap-ncat-2:7.92-1.el8.x86_64                                                                                  2/4 
  Verifying        : libpcap-14:1.9.1-5.el8.x86_64                                                                                  3/4 
  Verifying        : libibverbs-46.0-1.el8.1.x86_64                                                                                 4/4 
Installed products updated.

Installed:
  libibverbs-46.0-1.el8.1.x86_64     libpcap-14:1.9.1-5.el8.x86_64     nmap-2:7.92-1.el8.x86_64     nmap-ncat-2:7.92-1.el8.x86_64    

Complete!
[root@da2a7a3b1281 /]#
```

Confirm that `nmap` is now installed:

```console
[root@da2a7a3b1281 /]# nmap --version
Nmap version 7.92 ( https://nmap.org )
Platform: x86_64-redhat-linux-gnu
Compiled with: nmap-liblua-5.3.5 openssl-1.1.1k libz-1.2.11 libpcre-8.42 libpcap-1.9.1 nmap-libdnet-1.12 ipv6
Compiled without: libssh2
Available nsock engines: epoll poll select
[root@da2a7a3b1281 /]# 
```

Exit the container and use `podman commit` to create the image based on the modified container:

```console
[user@localhost ~]$ podman commit nmap-test nmap-test
Getting image source signatures
Copying blob d93817448019 skipped: already exists  
Copying blob fa2756e25f39 done  
Copying config fc1e3b460b done  
Writing manifest to image destination
fc1e3b460b347cb099f2503d22b067da95bce7d1d367b43861ac2e96b4075e93
[user@localhost ~]$
```

The new image can be seen from the command `podman images`:

```console
[user@localhost ~]$ podman images 
REPOSITORY                       TAG         IMAGE ID      CREATED        SIZE
localhost/nmap-test              latest      ede886c1cb62  2 minutes ago  506 MB
registry.access.redhat.com/ubi8  latest      86b358a425da  3 weeks ago    213 MB
```

When you start a container with this new image, nmap will be available.

```console
[user@localhost ~]$ podman run nmap-test:latest nmap --version
Nmap version 7.92 ( https://nmap.org )
Platform: x86_64-redhat-linux-gnu
Compiled with: nmap-liblua-5.3.5 openssl-1.1.1k libz-1.2.11 libpcre-8.42 libpcap-1.9.1 nmap-libdnet-1.12 ipv6
Compiled without: libssh2
Available nsock engines: epoll poll select
```