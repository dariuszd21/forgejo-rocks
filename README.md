# forgejo-rocks
Rocks (OCI images) for the Forgejo

# Installation

## Create a pod for forgejo sandbox
```shell
podman pod create -p 3000:3000 -p 2222:22 forgejo-dev
```

## Create postgresql database
> PostgreSQL here is ephemeral and data will not persist in case of upgrade of recreation (only configuration is persistent as it is a local file).

```shell
podman pull docker.io/library/postgres:16.4-alpine3.20
podman run --name forgejo-postgresql --pod forgejo-dev -v \
	"$PWD/my-postgres.conf":/etc/postgresql/postgresql.conf \
	-e POSTGRES_PASSWORD=<put-some-your-password-in-here> \
	-e POSTGRES_USER=<put-some-custom-user-in-here> \
	postgres:16.4-alpine3.20 \
	-c 'config_file=/etc/postgresql/postgresql.conf'
```

## Build rock
```shell
rockcraft pack
```

## Import image to podman
```shell
podman load --input forgejo_8.0.1_amd64.rock
podman tag localhost/8.0.1:latest localhost/forgejo:8.0.1
podman image rm localhost/8.0.1:latest
```

## Create peristent storage for the config and repositories

```shell
podman volume create forgejo-conf
podman volume create forgejo-data
```

## Run image in the pod
```shell
podman run --pod forgejo-dev --name forgejo -v forgejo-conf:/etc/forgejo -v forgejo-data:/var/lib/forgejo -ti localhost/forgejo:8.0.1
```
