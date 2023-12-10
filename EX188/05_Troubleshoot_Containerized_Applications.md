# Troubleshoot Containerized Applications

## Objectives

- Understand the description of application resources
- Get application [logs](#podman-logs)
- Inspect running applications
  - `podman inspect`
  - [`podman logs`](#podman-logs)
- Connecting to running containers
  - `podman exec`
  - `podman attach`
- Additional troubleshooting tips
  - Network issues
  - Running host commands inside container using [`nsenter`](#nsenter)
  - [`podman unshare`](#podman-unshare)

## podman logs

`podman logs` display the logs of one or more containers.  Syntax:

```bash
podman logs [options] container [container...]
```


## Additional Troubleshooting Tips

### Network Issues

For network issues, check the current port mappings by either:

```bash
podman ps -a
```

or
```bash
podman port [options] container [private-port[/proto]]
```

Check container running ports by running the `ss` command within the container:

```bash
ss -pant
```


### nsenter

Run host commands within the container which may not have the application installed by using `nsenter`:

```bash
nsenter [options] [program [arguments]]
```

For example:
```bash
nsenter -n -t PID
```

To find the PID of a running container, use the command:

```bash
podman inspect CONTAINERID --format '{{.State.Pid}}'
```


### podman unshare

`podman unshare` runs a command inside of a modified user namespace.  

`podman unshare` launches a process (by default, $SHELL) in a new user namespace. 

The user namespace is configured so that the invoking user's UID and primary GID appear to be UID 0 and GID 0, respectively.  

Any ranges which match that user and group in /etc/subuid and /etc/subgid are also mapped in as  themselves with the help of the newuidmap(1) and newgidmap(1) helpers.

podman unshare is useful for troubleshooting unprivileged operations and for manually clearing storage and other data related to images and containers.
