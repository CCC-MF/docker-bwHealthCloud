# Docker files to build images for bwHealthCloud Frontend and Backend

The purpose of this project is to provide docker images for [bwHealthCloud](https://www.telemedbw.de/projekte/bwhealthcloud).

## Deprecation and public archive

The purpose of this project was to provide basic docker support to bwHC packages.

Since there is a usable [pull request](https://github.com/KohlbacherLab/bwhc-frontend/pull/1)
and [docker image](https://github.com/CCC-MF/bwhc-frontend/pkgs/container/bwhc-frontend)
for frontend and some work is done based on this projects Dockerfile to create backend images,
this project will be deprecated and become a public archive, soon.

### Frontend
For frontend docker images, see: https://github.com/CCC-MF/bwhc-frontend/tree/docker-dev

Images will be build from `docker-dev` branch and rebased on current https://github.com/KohlbacherLab/bwhc-frontend.

### Backend
For backend docker images, see: https://github.com/KohlbacherLab/bwHC-REST-API-Gateway

## First steps

Make sure that latest release files are in the project directory, as docker build will pick them up without downloading them.

Currently frontend release file will be downloaded from [https://bwhealthcloud.de](https://bwhealthcloud.de/downloads/latest/bwhc-frontend.zip) which might be outdated.

Latest release files are available from [https://ibmi-intra.cs.uni-tuebingen.de](https://ibmi-intra.cs.uni-tuebingen.de/display/ZPM/bwHC+Prototype+Release+Notes) - login required.

## Build images

This project contains two files for use with frontend and backend `Frontend.Dockerfile` and `Backend.Dockerfile`.
Both of them use build arguments to customize build on build time.

### Frontend

To build the image for bwHealthCloud frontend, use the following command

```
docker build -t bwhc-frontend -f Frontend.Dockerfile .
```

This will create the image using default build arguments and mark it as `bwhc-frontend`. To customize the build, spezify custom values.

* `VERSION`: The version to be used. Current value `2209`. Required for use with local release zip file.
* `NUXT_HOST` and `NUXT_PORT`: Server configuration, see 2.3 of bwHC manual for more information.
* `BACKEND_PROTOCOL`, `BACKEND_HOSTNAME` and `BACKEND_PORT`: Backend access, see 2.4 of bwHC manual for more information.

e.g.:

```
docker build -t bwhc-frontend -f Frontend.Dockerfile --build-arg NUXT_PORT=1234 .
```

### Backend

To build the image for bwHealthCloud backend, use the following command

```
docker build -t bwhc-backend -f Backend.Dockerfile .
```

This will create the image with the default build arguments and mark it as `bwhc-backend`. To customize the build, specify custom values.

* `VERSION`: The version to be used. Current value `2209`
* `BWHC_BASE_DIR`: The directory to hold the application and config files.
  This value defaults to `/bwhc-backend`. Changing this value requires changing paths in example `docker-compose.yml`.

e.g.:

```
docker build -t bwhc-backend -f Backend.Dockerfile --build-arg BWHC_BASE_DIR=/opt/bwhc-backend .
```

## Run the images

Both images can be started like any other docker image. Make sure you provide required docker volumes to keep data after service recreation.

The backend image uses two volumes relative to `$BWHC_BASE_DIR` provided at build time: `$BWHC_BASE_DIR/data` and `$BWHC_BASE_DIR/hgnc_data`.

This project provides support for Docker Compose. So it is possible to build and run frontend and backend using Docker compose.

### Using configuration files as volume

To use custom logging configuration in `logback.xml` and/or custom `bwhcConnectorConfig.xml` for backend,
mount these files as readonly volumes or docker configuration to `$BWHC_BASE_DIR/logback.xml`
and `$BWHC_BASE_DIR/bwhcConnectorConfig.xml`.

```
docker run -v $PWD/bwhcConnectorConfig.xml:/bwhc-backend/bwhcConnectorConfig.xml:ro
```

Or using docker-compose liko so:

```
...

services:
    backend:
    #    ... other settings ...
    volumes:
        - "./bwhcConnectorConfig.xml:/bwhc-backend/bwhcConnectorConfig.xml:ro"

...
```

### Using Docker configuration

To use configuration files as docker config create a new docker config from configuration file.

```
docker config create bwhcConnectorConfig bwhcConnectorConfig.xml
```

Using docker-compose, change docker-compose file to contain entries like so.

```
configs:
    bwhcConnectorConfig:
        external: true

...

services:
    backend:
    #    ... other settings ...
    configs:
        - source: bwhcConnectorConfig
          target: /bwhc-backend/bwhcConnectorConfig.xml

...
```

## Limitations and final notes

The image for bwHealthCloud Frontend starts as expected, using configured build arguments and environment variables at build time.
Since frontends access to the backend is configured at build time, it is not required to provide environment variables at runtime.

The bwHealthCloud Backend image starts using play start script provided by application.
I have not tested if there are any issues running the backend in addition to login.
