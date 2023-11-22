# Basic Podman Commands

## Objectives

List of basic podman commands: 

- podman -v
- podman pull 
- podman images / podman image list
- [podman run](#podman-run)
- podman stop
  - podman stop --all
- podman kill
- podman pause
- podman unpause
- podman restart
- podman rm
  - podman rm --all
- podman ps
- podman ps -a
- [podman network](#podman-network)
- podman exec 
- podman cp
- podman inspect
  - podman inspect --format '{{.State.Pid}}' CONTAINER
  - podman inspect --format '{{.State.Status}}' CONTAINER
  - podman inspect --format '{{.State.Running}}' CONTAINER``


## podman run

For full list of options, refer to the man page of `podman run`:

```bash
man podman-run
```

Common options:

```quote
-d
-it
--rm
-p    forwards a port, format HOST_PORT:CONTAINER_PORT
-e
--name
--net
```

## podman network

`podman network` manages networks for podman.  Refer to the man page for more details:

```bash
man podman-network
```

Common subcommands:
- create
- ls
- inspect
- connect
- disconnect
- prune
- rm

Use the option --net to connect to a specific network when running containers, otherwise the container will be using the default `podman` network.