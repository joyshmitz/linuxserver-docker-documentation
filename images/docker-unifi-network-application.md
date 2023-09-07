---
title: unifi-network-application
---
<!-- DO NOT EDIT THIS FILE MANUALLY  -->
<!-- Please read the https://github.com/linuxserver/docker-unifi-network-application/blob/main/.github/CONTRIBUTING.md -->

# [linuxserver/unifi-network-application](https://github.com/linuxserver/docker-unifi-network-application)

[![Scarf.io pulls](https://scarf.sh/installs-badge/linuxserver-ci/linuxserver%2Funifi-network-application?color=94398d&label-color=555555&logo-color=ffffff&style=for-the-badge&package-type=docker)](https://scarf.sh/gateway/linuxserver-ci/docker/linuxserver%2Funifi-network-application)
[![GitHub Stars](https://img.shields.io/github/stars/linuxserver/docker-unifi-network-application.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&logo=github)](https://github.com/linuxserver/docker-unifi-network-application)
[![GitHub Release](https://img.shields.io/github/release/linuxserver/docker-unifi-network-application.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&logo=github)](https://github.com/linuxserver/docker-unifi-network-application/releases)
[![GitHub Package Repository](https://img.shields.io/static/v1.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=linuxserver.io&message=GitHub%20Package&logo=github)](https://github.com/linuxserver/docker-unifi-network-application/packages)
[![GitLab Container Registry](https://img.shields.io/static/v1.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=linuxserver.io&message=GitLab%20Registry&logo=gitlab)](https://gitlab.com/linuxserver.io/docker-unifi-network-application/container_registry)
[![Quay.io](https://img.shields.io/static/v1.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=linuxserver.io&message=Quay.io)](https://quay.io/repository/linuxserver.io/unifi-network-application)
[![Docker Pulls](https://img.shields.io/docker/pulls/linuxserver/unifi-network-application.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=pulls&logo=docker)](https://hub.docker.com/r/linuxserver/unifi-network-application)
[![Docker Stars](https://img.shields.io/docker/stars/linuxserver/unifi-network-application.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=stars&logo=docker)](https://hub.docker.com/r/linuxserver/unifi-network-application)
[![Jenkins Build](https://img.shields.io/jenkins/build?labelColor=555555&logoColor=ffffff&style=for-the-badge&jobUrl=https%3A%2F%2Fci.linuxserver.io%2Fjob%2FDocker-Pipeline-Builders%2Fjob%2Fdocker-unifi-network-application%2Fjob%2Fmain%2F&logo=jenkins)](https://ci.linuxserver.io/job/Docker-Pipeline-Builders/job/docker-unifi-network-application/job/main/)

The [Unifi-network-application](https://ui.com/) software is a powerful, enterprise wireless software engine ideal for high-density client deployments requiring low latency and high uptime performance.

## Supported Architectures

We utilise the docker manifest for multi-platform awareness. More information is available from docker [here](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md#manifest-list) and our announcement [here](https://blog.linuxserver.io/2019/02/21/the-lsio-pipeline-project/).

Simply pulling `lscr.io/linuxserver/unifi-network-application:latest` should retrieve the correct image for your arch, but you can also pull specific arch images via tags.

The architectures supported by this image are:

| Architecture | Available | Tag |
| :----: | :----: | ---- |
| x86-64 | ✅ | amd64-\<version tag\> |
| arm64 | ✅ | arm64v8-\<version tag\> |
| armhf | ❌ | |

## Application Setup

### This container requires an external mongodb database instance.

The web UI is at https://ip:8443, setup with the first run wizard.

### Setting Up Your External Database

Formally only mongodb 3.6 through 4.4 are supported, however, it has been reported that newer versions will work. If you choose to use a newer version be aware that you will not be operating a supported configuration.

**Make sure you pin your database image version and do not use `latest`, as mongodb does not support automatic upgrades between major versions.**

If you are using the [official mongodb container](https://hub.docker.com/_/mongo/), you can create your databases using an `init-mongo.js` file with the following contents:

```js
db.getSiblingDB("MONGO_DBNAME").createUser({user: "MONGO_USER", pwd: "MONGO_PASS", roles: [{role: "readWrite", db: "MONGO_DBNAME"}]});
db.getSiblingDB("MONGO_DBNAME_stat").createUser({user: "MONGO_USER", pwd: "MONGO_PASS", roles: [{role: "readWrite", db: "MONGO_DBNAME_stat"}]});
```

Being sure to replace the placeholders with the same values you supplied to the Unifi container, and mount it into your *mongodb* container.

```yaml
volumes:
  - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
```

*Note that the init script method will only work on first run. If you start the mongodb container without an init script it will generate test data automatically and you will have to manually create your databases, or restart with a clean `/data/db` volume and an init script mounted.*

You can also run the commands directly against the database using either `mongo` (< 6.0) or `mongosh` (>= 6.0).

### Device Adoption

For Unifi to adopt other devices, e.g. an Access Point, it is required to change the inform IP address. Because Unifi runs inside Docker by default it uses an IP address not accessible by other devices. To change this go to Settings > System > Advanced and set the Inform Host to a hostname or IP address accessible by your devices. Additionally the checkbox "Override" has to be checked, so that devices can connect to the controller during adoption (devices use the inform-endpoint during adoption).

**Please note, Unifi change the location of this option every few releases so if it's not where it says, search for "Inform" or "Inform Host" in the settings.**

In order to manually adopt a device take these steps:

```
ssh ubnt@$AP-IP
set-inform http://$address:8080/inform
```

The default device password is `ubnt`. `$address` is the IP address of the host you are running this container on and `$AP-IP` is the Access Point IP address.

When using a Security Gateway (router) it could be that network connected devices are unable to obtain an ip address. This can be fixed by setting "DHCP Gateway IP", under Settings > Networks > network_name, to a correct (and accessible) ip address.

### Migration From [Unifi-Controller](https://github.com/linuxserver/docker-unifi-controller)

If you were using the `mongoless` tag for the Unifi Controller container, you can switch directly to the Unifi Network Application container without needing to perform any migration steps.

**You cannot perform an in-place upgrade from an existing Unifi-Controller container, you must run a backup and then a restore.**

The simplest migration approach is to take a full backup of your existing install, including history, from the Unifi-Controller web UI, then shut down the old container.

You can then start up the new container with a clean `/config` mount (and a database container configured), and perform a restore using the setup wizard.

### Strict reverse proxies

This image uses a self-signed certificate by default. This naturally means the scheme is `https`.
If you are using a reverse proxy which validates certificates, you need to [disable this check for the container](https://docs.linuxserver.io/faq#strict-proxy).

## Usage

To help you get started creating a container from this image you can either use docker-compose or the docker cli.

### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
version: "2.1"
services:
  unifi-network-application:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi-network-application
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - MONGO_USER=unifi
      - MONGO_PASS=
      - MONGO_HOST=unifi-db
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi
      - MEM_LIMIT=1024 #optional
      - MEM_STARTUP=1024 #optional
    volumes:
      - /path/to/data:/config
    ports:
      - 8443:8443
      - 3478:3478/udp
      - 10001:10001/udp
      - 8080:8080
      - 1900:1900/udp #optional
      - 8843:8843 #optional
      - 8880:8880 #optional
      - 6789:6789 #optional
      - 5514:5514/udp #optional
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=unifi-network-application \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e MONGO_USER=unifi \
  -e MONGO_PASS= \
  -e MONGO_HOST=unifi-db \
  -e MONGO_PORT=27017 \
  -e MONGO_DBNAME=unifi \
  -e MEM_LIMIT=1024 `#optional` \
  -e MEM_STARTUP=1024 `#optional` \
  -p 8443:8443 \
  -p 3478:3478/udp \
  -p 10001:10001/udp \
  -p 8080:8080 \
  -p 1900:1900/udp `#optional` \
  -p 8843:8843 `#optional` \
  -p 8880:8880 `#optional` \
  -p 6789:6789 `#optional` \
  -p 5514:5514/udp `#optional` \
  -v /path/to/data:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/unifi-network-application:latest

```

## Parameters

Docker images are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

### Ports (`-p`)

| Parameter | Function |
| :----: | --- |
| `8443` | Unifi web admin port |
| `3478/udp` | Unifi STUN port |
| `10001/udp` | Required for AP discovery |
| `8080` | Required for device communication |
| `1900/udp` | Required for `Make controller discoverable on L2 network` option |
| `8843` | Unifi guest portal HTTPS redirect port |
| `8880` | Unifi guest portal HTTP redirect port |
| `6789` | For mobile throughput test |
| `5514/udp` | Remote syslog port |

### Environment Variables (`-e`)

| Env | Function |
| :----: | --- |
| `PUID=1000` | for UserID - see below for explanation |
| `PGID=1000` | for GroupID - see below for explanation |
| `TZ=Etc/UTC` | specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `MONGO_USER=unifi` | Mongodb Username. Only evaluated on first run. |
| `MONGO_PASS=` | Mongodb Password. Only evaluated on first run. |
| `MONGO_HOST=unifi-db` | Mongodb Hostname. Only evaluated on first run. |
| `MONGO_PORT=27017` | Mongodb Port. Only evaluated on first run. |
| `MONGO_DBNAME=unifi` | Mongodb Database Name (stats DB is automatically suffixed with `_stat`). Only evaluated on first run. |
| `MEM_LIMIT=1024` | Optionally change the Java memory limit (in Megabytes). Set to `default` to reset to default |
| `MEM_STARTUP=1024` | Optionally change the Java initial/minimum memory (in Megabytes). Set to `default` to reset to default |

### Volume Mappings (`-v`)

| Volume | Function |
| :----: | --- |
| `/config` | All Unifi data stored here |

#### Miscellaneous Options

| Parameter | Function |
| :-----:   | --- |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

Will set the environment variable `PASSWORD` based on the contents of the `/run/secrets/mysecretpassword` file.

## Umask for running applications

For all of our images we provide the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags), permissions issues can arise between the host OS and the container, we avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```bash
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

## Docker Mods

[![Docker Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=unifi-network-application&query=%24.mods%5B%27unifi-network-application%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=unifi-network-application "view available mods for this container.") [![Docker Universal Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=universal&query=%24.mods%5B%27universal%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=universal "view available universal mods.")

We publish various [Docker Mods](https://github.com/linuxserver/docker-mods) to enable additional functionality within the containers. The list of Mods available for this image (if any) as well as universal mods that can be applied to any one of our images can be accessed via the dynamic badges above.

## Support Info

* Shell access whilst the container is running:
  * `docker exec -it unifi-network-application /bin/bash`
* To monitor the logs of the container in realtime:
  * `docker logs -f unifi-network-application`
* Container version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' unifi-network-application`
* Image version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/unifi-network-application:latest`

## Versions

* **05.09.23:** - Initial release.