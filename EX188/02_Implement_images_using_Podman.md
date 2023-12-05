# Implement images using Podman

## Objectives

- [Image Registries](03_Manage_Images.md#image-registries)
- [`skopeo` command](03_Manage_Images.md#skopeo-command)
- [Containerfile](#containerfile)
  - Understand and use [FROM](#from) (the concept of a base image) instruction.
  - Understand and use [RUN](#run) instruction.
  - Understand and use [ADD](#add) instruction.
  - Understand and use [COPY](#copy) instruction.
    - [Multistage builds](#multistage-builds)
  - Understand the difference between ADD and COPY instructions.
  - Understand and use [WORKDIR](#workdir) and [USER](#user) instructions.
  - Understand security-related topics.
  - Understand the differences and applicability of [CMD vs. ENTRYPOINT](#entrypoint-vs-cmd) instructions.
  - Understand [ENTRYPOINT](#entrypoint) instruction with param.
  - Understand when and how to [expose](#expose) ports from a Containerfile.
  - Understand and use [environment variables](#env) inside images.
  - Understand [ENV](#env) instruction.
    - [ARG](#arg) instruction
  - Understand container [volume](#volume).
  - [Mount a host directory as a data volume](#mount-a-host-directory-as-a-data-volume)
    - Volumes: data mounts managed by podman
    - Bind mounts: data mounts managed by user
  - Understand security and permissions requirements related to this approach.
  - Understand the lifecycle and cleanup requirements of this approach.


## Containerfile

Containerfile is similar to Dockerfile in the docker world.  It provides a set of instructions to build a container image.  

Refer to the man page for `containerfile`:

```bash
man containerfile
```

Excerpt:

```console
    The Containerfile is a configuration file that automates the  steps  of
    creating  a container image. It is similar to a Makefile. Container en‐
    gines (Podman, Buildah, Docker) read instructions from  the  Container‐
    file  to  automate  the steps otherwise performed manually to create an
    image. To build an image, create a file called Containerfile.
```

### FROM

Sets the base image from which to build the container image.

```console
FROM    registry.access.redhat.com/ubi8
```


### RUN

RUN has two forms:

```console
# the command is run in a shell - /bin/sh -c
RUN <command>

# Executable form
RUN ["executable", "param1", "param2"]
```

### ADD

Copies new files, directories or remote file URLs to the filesystem of the container at specified destination path.

Destination path is the absolute path, or path relative to WORKDIR, into which the source is copied inside the target container.

If source is a local file in a recognized compression format (tar, gzip, bzip2, etc) then it is unpacked at the specified destination in the container's filesystem.  Note that only local compressed files will be unpacked, the URL download and archive unpacking features cannot be used together.
         
All new directories are created with mode 0755 and with the uid and gid of 0.


### COPY

```console
COPY [--chown=<user>:<group>] [--chmod=<mode>] <src> <dest>
```

The COPY instruction copies new files from `<src>` and adds them to the filesystem of the container at path. 

The `<src>` must be the path to a file or directory relative to the source directory that is being built (the context of the build) or a remote file URL. 

The `<dest>` is an absolute path, or a path relative to WORKDIR, into which the source will be copied inside the target container. 

**If you COPY an archive file it will land in the container exactly as it appears in the build context without any attempt to unpack it.**

All new files and directories are created with mode 0755 and with the uid and gid of 0.


#### Multistage Builds

The optional flag --from=name of the COPY instruction can be used to copy files from a named previous build stage. 

It changes the context of `<src>` from the build context to the named build stage.

Example:

```console
# Containerfile

# Build Stage
FROM some-build-image as builder

# Install build dependencies
RUN yum install -y build-essential

# Copy source code
COPY . /app

# Build the application
RUN make

# Final Stage
FROM some-base-image

# Copy only the built artifacts from the build stage
COPY --from=builder /app/bin/app /usr/local/bin/app

# Set the command to run the application
CMD ["app"]
```


### WORKDIR

Sets the working directory.  The instructions in the Containerfile which follows executes in this working directory.


### USER

Sets the username or UID used for running subsequent commands.  

Until the USER instruction is set, instructions will be run as root. 

The USER instruction can be used any number of times in a Containerfile, and will only affect subsequent commands.

Podman maps users inside of the container to unprivileged users on the host system by using subordinate ID ranges.  For more information:

```bash
man subuid
man subgid
```

Use `usermod` command to generate `subuid` and `subgid` ranges.  Subsequently run `podman system migrate` to make the ids effective.  

Use `podman top` to see information on running processes:
- `user`
- `huser`



### ENTRYPOINT

When you specify an ENTRYPOINT, the whole container runs as if it was only that executable.  

The ENTRYPOINT instruction adds an entry command that is not overwritten when arguments are passed to podman run. 

ENTRYPOINT has two forms:

```console
# executable form
ENTRYPOINT ["executable", "param1", "param2]

# run command in a shell - /bin/sh -c
ENTRYPOINT command param1 param2
```


### ENTRYPOINT vs CMD

ENTRYPOINT behaviour is different from that of CMD. CMD allows arguments to be passed to the entrypoint, for instance:

```bash
podman run <image> -d
``` 

This passes the -d argument to the ENTRYPOINT.  

Specify parameters either in the ENTRYPOINT JSON array (as in the preferred exec form above), or by using a CMD statement.  

Parameters in the ENTRYPOINT are not overwritten by the podman run  arguments.   

Parameters specified via CMD are overwritten by podman run arguments.  


### EXPOSE

```console
-- EXPOSE <port> [<port>...]
```

The EXPOSE instruction informs the container engine that the container listens on the specified network ports at runtime. 

The container engine uses this information to interconnect containers using links and to set up port redirection on the host system.


### ENV

```console
-- ENV <key> <value>
```

The ENV instruction sets the environment variable to the value `<value>`. 

This value is pased to all future `RUN`, `ENTRYPOINT`, and `CMD` instructions. 

The environment variables that are set with ENV perist when a container is run from the resulting image. 

Use `podman insect` to inspect these values, and change them using `podman run --env key =<value>`.


### ARG

Excerpt from man page:

```console
-- ARG [=]
```

The ARG instruction defines a variable that users can pass at build-time to the builder with the `podman build` and `buildah build` commands using the `--build-arg <varname>=<value>` flag. 

If a user specifies a build argument that was not defined in the Containerfile, the build outputs a warning.

The Containerfile author can define a single variable by specifying ARG once or many variables by specifying ARG more than once. For example, a valid Containerfile:

```console
  FROM busybox
  ARG user1
  ARG buildno
  ...
```
A Containerfile author may optionally specify a default value for an ARG instruction:

```console
  FROM busybox
  ARG user1=someuser
  ARG buildno=1
  ...
```
If an ARG value has a default and if there is no value passed at build-time, the builder uses the default.

An ARG variable definition comes into effect from the line on which it is defined in the Containerfile not from the argument's use on the command-line or elsewhere.  For example, consider this Containerfile:

```console
  1 FROM busybox
  2 USER ${user:-some_user}
  3 ARG user
  4 USER $user
  ...
```

A user builds this file by calling:

```bash
podman build --build-arg user=what_user Containerfile
```

The USER at line 2 evaluates to `some_user` as the user variable is defined on the subsequent line 3. 

The USER at line 4 evaluates to `what_user` as user is defined and the what_user value was passed on the command line. 

Prior to its  definition by an ARG instruction, any use of a variable results in an empty string.


### VOLUME

```console
-- VOLUME ["/data"]
```

The VOLUME instruction creates a mount point with the specified name and marks it as holding externally-mounted volumes from the native host or from other containers.

To remove unused volumes, use `podman volume prune`

To check which host folder is being mapped to the volume, use `podman volume inspect`

To create a named volume, use `podman volume create VOLUME`

To list existing volumes, run `podman volume ls`


### Mount a Host Directory as a Data Volume

When running `podman`, use the `--volume` or `-v` option to mount a host directory as a data volume.  This will create a bind mount or volume mount:  

```console
--volume, -v=[[SOURCE-VOLUME|HOST-DIR:]CONTAINER-DIR[:OPTIONS]]

  Create a bind mount. 
  If -v /HOST-DIR:/CONTAINER-DIR is specified, Podman bind mounts /HOST-DIR from the host into /CONTAINER-DIR in the Podman container. 
  
  Similarly, -v SOURCE-VOLUME:/CONTAINER-DIR mounts the named  volume from the host into the container. If no such named volume
  exists, Podman creates one. 
  
  If no source is given, the volume  is  created as an anonymously named volume with a randomly generated name, and
  is removed when the container is removed via the --rm flag or the  podman rm --volumes command.

  The OPTIONS is a comma-separated list such as:
    • rw|ro
    • z|Z
```

For bind mount, we need to take care of the SELinux context and the `z` or `Z` option will take care of this:
- `z` gives all containers access
- `Z` exclusively gives this container access

For volume mount, podman manages the volume and takes care of the SELinux contexts.  