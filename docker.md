# Docker
[Docker](https://wiki.archlinux.org/title/docker) is a container engine.

# Docker Compose

Docker Compose is an alternate to the Docker CLI that offloads configuration to YAML files.
Compose allows the user to create and destroy multiple Docker resources at once.

```bash
pacman -S docker-compose
```

# Deploy a Registry Server

Sourced from [Deploy a Registry Server](https://docs.docker.com/registry/deploying/).
This `docker-compose.yaml` file mounts an NFS share from a remote host to create a
Docker registry.

```yaml
services:
    registry:
        restart: always
        image: registry:2
        ports:
            - 5000:5000
        volumes:
            - type: volume
              source: nfs-mount
              target: /var/lib/registry
    test:
        image: debian
        tty: true
        volumes:
            - type: volume
              source: nfs-mount
              target: /var/lib/registry
volumes:
    nfs-mount:
        driver: local
        driver_opts:
            type: nfs
            o: "addr=hostname,rw"
            device: ":/mnt/.../docker-registry"

```
