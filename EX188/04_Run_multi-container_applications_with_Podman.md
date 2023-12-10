# Run Multi-container Applications with Podman

## Objectives

- Create application stacks
  - `podman compose`
    - up
    - down
    - start
    - stop
- Understand container dependencies
  - `depends_on:`
- Working with environment variables
- Working with secrets
- Working with volumes
- Working with configuration


## `podman compose`

```bash
podman-compose -h
```

`podman compose` files are in YAML format.  

Important compose file sections:
- version
- services
  - image
  - container_name
  - environment
  - volumes
  - networks
  - ports
  - depends_on
- networks
- volumes
- [configs](#configs)
- [secrets](#secrets)


Example compose files can be found in the directory `/usr/share/doc/podman-compose/examples/`:

```console
---
volumes:
  db_data:
services:
  wordpress:
    image: docker.io/library/wordpress:latest
    ports:
      - 8080:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=password
      - WORDPRESS_DB_NAME=wordpress
  db:
    image: docker.io/library/mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=password
```

## Example Illustrating configs, secrets, volumes

Refer to this [page](https://github.com/compose-spec/compose-spec/blob/master/spec.md) for a detailed explanation illustrating the differences.


### configs

`configs` are similar to volumes, in that they are files mounted into the container. 

`configs` are used to store configuration files and settings for services in your compose stack for non-sensitive configuration data that needs to be **shared across multiple services**.


### secrets

`secrets` are a specific type of `configs` data for sensitive data. 

`secrets` are designed to manage sensitive data such as passwords, such information should not be exposed or stored in the compose file directly.  

`secrets` are **specific to individual services**.

Refer to the manual page for `podman-run` for more details:

```config
When secrets are specified as type mount, the secrets are copied and mounted into the container when a container is created.  

When secrets are specified as type env, the secret is  set  as  an  environment variable  within  the  container.   

Secrets are written in the container at the time of container creation, and modifying the secret using podman secret commands after the container is created affects the secret inside the container.
```


## References

[The Compose Specification](https://github.com/compose-spec/compose-spec/blob/master/spec.md)
[Compose Spec](https://github.com/compose-spec/compose-spec)
