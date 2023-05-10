# Docker
[Docker](https://wiki.archlinux.org/title/docker) is a container engine.

```bash
docker run -it --rm --name debby debian bash
docker exec -it debby bash
```

# Docker Compose

Docker Compose is an alternate to the Docker CLI that offloads configuration to YAML files.
Compose allows the user to create and destroy multiple Docker resources at once.

```bash
pacman -S docker-compose
```

Every Docker Compose project should have its own folder.
Inside is a single file called `docker-compose.yaml`.

```bash
docker compose up                # Start containers.
docker compose down              # Stop containers.
docker compose down --volumes    # Also removes all unused volumes.
```

# Deploy a Registry Server

Sourced from [Deploy a Registry Server](https://docs.docker.com/registry/deploying/).
This `docker-compose.yaml` file mounts an NFS share from a remote host to create a
Docker registry.

Be aware that the registry server is not transparent.
`Dockerfile`'s which want to reference images hosted here must do so explicitly using
the first part of the image identifier `<host:ip>/<image>:<tag>`.
If ommitted, the global Docker Hub registry at `docker.io` is assumed.

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

## Upload Images

By default, the registry server is not a pass-through to
[hub.docker.com](hub.docker.com)/[docker.io](docker.io).
It must be manually populated with the images you want to host.

```bash
docker pull ubuntu:16.04
docker tag ubuntu:16.04 host:ip/ubuntu:16.04
docker push host:ip/ubuntu:16.04
```
