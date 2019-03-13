# docker-compose-postgresql-demo

## Overview

The following diagram shows the relationship of the docker containers in this docker composition.

![Image of architecture](doc/img-architecture/architecture.png)

This docker formation brings up the following docker containers:

1. *[postgres](https://hub.docker.com/_/postgres)*
1. *[dockage/phppgadmin](https://hub.docker.com/r/dockage/phppgadmin)*
1. *[senzing/python-demo](https://github.com/Senzing/docker-python-demo)*

Also shown in the demonstration are commands to run the following Docker images:

1. *[senzing/g2loader](https://github.com/Senzing/docker-g2loader)* in [Run G2Loader.py](#run-g2loaderpy)
1. *[senzing/g2command](https://github.com/Senzing/docker-g2command)* in [Run G2Command.py](#run-g2commandpy)

### Contents

1. [Preparation](#preparation)
    1. [Clone repository](#clone-repository)
    1. [Create SENZING_DIR](#create-senzing_dir)
    1. [Prerequisite Software](#prerequisite-software)
1. [Using docker-compose](#using-docker-compose)
    1. [Build docker images](#build-docker-images)
    1. [Configuration](#configuration)
    1. [Launch docker formation](#launch-docker-formation)
    1. [Initialize database](#initialize-database)
    1. [Run G2Loader.py](#run-g2loaderpy)
    1. [Run G2Command.py](#run-g2commandpy)
1. [Cleanup](#cleanup)

## Preparation

### Clone repository

1. Set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-postgres-demo
    ```

   Then follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md).

1. After the repository has been cloned, be sure the following are set:

    ```console
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

### Create SENZING_DIR

If you do not already have an `/opt/senzing` directory on your local system, visit
[HOWTO - Create SENZING_DIR](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/create-senzing-dir.md).

### Prerequisite software

The following software programs need to be installed.

#### docker

1. [Install docker](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker.md)
1. Test

    ```console
    sudo docker --version
    sudo docker run hello-world
    ```

#### docker-compose

1. [Install docker-compose](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker-compose.md)
1. Test

    ```console
    sudo docker-compose --version
    ```

## Using docker-compose

### Build docker images

1. Build docker images.

    ```console
    export BASE_IMAGE=senzing/python-postgresql-base

    sudo docker build \
      --tag ${BASE_IMAGE} \
      https://github.com/senzing/docker-python-postgresql-base.git

    sudo docker build \
      --tag senzing/python-demo \
      --build-arg BASE_IMAGE=${BASE_IMAGE} \
      https://github.com/senzing/docker-python-demo.git

    sudo docker build \
      --tag senzing/g2loader \
      --build-arg BASE_IMAGE=${BASE_IMAGE} \
      https://github.com/senzing/docker-g2loader.git

    sudo docker build \
      --tag senzing/g2command \
      --build-arg BASE_IMAGE=${BASE_IMAGE} \
      https://github.com/senzing/docker-g2command.git
    ```

### Configuration

- **SENZING_DIR** -
  Path on the local system where
  [Senzing_API.tgz](https://s3.amazonaws.com/public-read-access/SenzingComDownloads/Senzing_API.tgz)
  has been extracted.
  See [Create SENZING_DIR](#create-senzing_dir).
  No default.
  Usually set to "/opt/senzing".
- **POSTGRES_DB** -
  The database to create upon first invocation. Default: "G2".
- **POSTGRES_PASSWORD** -
  The password for the the database "postgres" user name.
  Default: "postgres"
- **POSTGRES_STORAGE** -
  Path on local system where the database files are stored.
  Default: "/storage/docker/senzing/docker-compose-postgresql-demo"

### Launch docker formation

1. Launch docker-compose formation.  Example:

    ```console
    cd ${GIT_REPOSITORY_DIR}

    export SENZING_DIR=/opt/senzing
    export POSTGRES_DB=G2
    export POSTGRES_PASSWORD=postgres
    export POSTGRES_STORAGE=/storage/docker/senzing/docker-compose-postgresql-demo

    sudo docker-compose up
    ```
    
    **Note:** `senzing-app` errors will be seen in the log until the database has been initialized in the next step.

### Initialize database

1. The database will be initialized using phpPgAdmin at [localhost:8080](http://localhost:8080).
   Login to phpPgAdmin with Username: postgres and Password: value of `POSTGRES_PASSWORD`.
1. In the left-hand navigation, highlight "G2" database.
1. Click "SQL" tab.
1. Click "Browse..." button and locate `/opt/senzing/g2/data/g2core-schema-postgresql-create.sql`
1. Click "Execute" button.
1. TODO: After the schema is loaded, the demonstration python/Flask app will be available at
   [localhost:5000](http://localhost:5000).

### Run G2Loader.py

In a separate terminal window:

1. Determine docker network. Example:

    ```console
    sudo docker network ls

    # Choose value from NAME column of docker network ls
    export SENZING_NETWORK=nameofthe_network
    ```

1. Run `docker` command. Example:

    ```console
    export DATABASE_PROTOCOL=postgresql
    export DATABASE_USERNAME=postgres
    export DATABASE_PASSWORD=postgres
    export DATABASE_HOST=senzing-postgres
    export DATABASE_PORT=5432
    export DATABASE_DATABASE=G2

    export SENZING_DATABASE_URL="${DATABASE_PROTOCOL}://${DATABASE_USERNAME}:${DATABASE_PASSWORD}@${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_DATABASE}"
    export SENZING_DIR=/opt/senzing

    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${SENZING_NETWORK} \
      --env SENZING_DATABASE_URL="${SENZING_DATABASE_URL}" \
      senzing/g2loader \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

### Run G2Command.py

In a separate terminal window:

1. Determine docker network. Example:

    ```console
    sudo docker network ls

    # Choose value from NAME column of docker network ls
    export SENZING_NETWORK=nameofthe_network
    ```

1. Run `docker` command. Example:

    ```console
    export DATABASE_PROTOCOL=postgresql
    export DATABASE_USERNAME=postgres
    export DATABASE_PASSWORD=postgres
    export DATABASE_HOST=senzing-postgres
    export DATABASE_PORT=5432
    export DATABASE_DATABASE=G2

    export SENZING_DATABASE_URL="${DATABASE_PROTOCOL}://${DATABASE_USERNAME}:${DATABASE_PASSWORD}@${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_DATABASE}"
    export SENZING_DIR=/opt/senzing

    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${SENZING_NETWORK} \
      --env SENZING_DATABASE_URL="${SENZING_DATABASE_URL}" \
      senzing/g2command
    ```

## Cleanup

In a separate terminal window:

1. Run `docker-compose` command.

    ```console
    cd ${GIT_REPOSITORY_DIR}
    sudo docker-compose down
    ```

1. Delete database storage.

    ```console
    sudo rm -rf ${POSTGRES_STORAGE}
    ```

1. Delete SENZING_DIR.

    ```console
    sudo rm -rf ${SENZING_DIR}
    ```

1. Delete git repository.

    ```console
    sudo rm -rf ${GIT_REPOSITORY_DIR}
    ```
