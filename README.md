# Supported tags and respective `Dockerfile` links

-	[`latest` (*h2/Dockerfile*)](https://github.com/GeoCat/geonetwork-2/blob/9ab31c98e7006658e7381924423263824f834e5c/h2/Dockerfile)
-	[`postgres` (*postgres/Dockerfile*)](https://github.com/GeoCat/geonetwork-2/blob/9ab31c98e7006658e7381924423263824f834e5c/h2/Dockerfile)

# What is GeoNetwork?

GeoNetwork is a catalog application to __manage spatially referenced resources__. It provides powerful __metadata editing__ and __search__ functions as well as an interactive __web map viewer__.

At present the project is widely used as the basis of __Spatial Data Infrastructures__ all around the world.

__Disclaimer:__ This image ships version 2x of GeoNetwork, which is __unsupported__. Only use this if you want to try the _old_ version of geonetwork (because ). For the latest version, please search for the official docker images.

![logo](https://raw.githubusercontent.com/GeoCat/geonetwork-2/master/logo.png)

# How to use this image
## Start geonetwork
This command will start a debian-based container, running a Tomcat web server, with a geonetwork war deployed on the server:
```console
$ docker run --name some-geonetwork -d doublebyte/geonetwork-2
```
## Publish port
Geonetwork listens on port `8080`. If you want to access the container at the host, __you must publish this port__. For instance, this, will redirect all the container traffic on port 8080, to the same port on the host:
```console
$ docker run --name some-geonetwork -d -p 8080:8080 doublebyte/geonetwork-2
```
Then, if you are running docker on Linux, you may access geonetwork at http://localhost:8080/geonetwork. Otherwise, replace `localhost` by the address of your docker machine.

## Set the data directory
The data directory is the location on the file system where the catalog stores much of its custom configuration and uploaded files. It is also where it stores a number of support files, used for various purposes (e.g.: Lucene index, spatial index, thumbnails).

By default, geonetwork sets the data directory on `/usr/local/tomcat/webapps/geonetwork/WEB-INF/data`, but you may override this value by injecting an environment variable into the container:
- `-e DATA_DIR=...` (defaults to `/usr/local/tomcat/webapps/geonetwork/WEB-INF/data`)

For instance, to set the data directory to `/var/lib/geonetwork_data`:
```console
$ docker run --name some-geonetwork -d -p 8080:8080 -e DATA_DIR=/var/lib/geonetwork_data doublebyte/geonetwork-2
```
## Persist data
If you want the data directory to live beyond restarts, or even destruction of the container, you can mount a directory from the docker engine's host into the container.
- `-v <host path>:<data directory>`

For instance this, will mount the host directory `/host/geonetwork-docker` into `/var/lib/geonetwork_data` on the container:
```console
$ docker run --name some-geonetwork -d -p 8080:8080 -e DATA_DIR=/var/lib/geonetwork_data -v /host/geonetwork-docker:/var/lib/geonetwork_data doublebyte/geonetwork-2
```
## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `geonetwork`:

```yaml
# GeoNetwork
#
# Access via "http://localhost:8080/geonetwork" (or "http://$(docker-machine ip):8080/geonetwork" if using docker-machine)
#
# Default user: admin
# Default password: admin

version: '2'
services:

    geonetwork:
      image: doublebyte/geonetwork-2
      ports:
          - 8080:8080
      environment:
          DATA_DIR: /var/lib/geonetwork_data
      volumes:
         - "/host/geonetwork-docker:/var/lib/geonetwork_data"
```

Run `docker-compose up`, wait for it to initialize completely, and visit `http://localhost:8080/geonetwork` or `http://host-ip:8080/geonetwork`.

## Default credentials
After installation a default user with name `admin` and password `admin` is created. Use this credentials to start with. It is recommended to update the default password after installation.

# Image Variants

The `geonetwork` images come in many flavors, each designed for a specific use case.

## `geonetwork-2:<version>`

This is the defacto image. If you are unsure about what your needs are, you probably want to use this one. It is designed to be used both as a throw away container (mount your source code and start the container to start your app), as well as the base to build other images off of.

By default, an H2 database is configured and created when the application first starts. If you are interested in a database engine other than H2, please have a look at other image variants.

## `geonetwork-2:postgres`

This image gives support for using [PostgreSQL](https://www.postgresql.org/) as database engine for geonetwork. When you start the container, a database is created, and it is populated by geonetwork, once it starts.

Please note that this image __does not ship__ the postgres database server itself, but it gives you the option to link to a container running postgres, or to connect to a postgres instance using its ip address. If you are looking for a self-contained installation of geonetwork, __including the database engine__, please have a look at the default image variant.

In order to setup the connection from geonetwork, you __must__ inject the following variables into the container:
- `POSTGRES_DB_USERNAME`: postgres user on your database server (must have permission to create databases)
- `POSTGRES_DB_PASSWORD`: postgres password on your database server

If your postgres instance is listening on a non-standard port, you must also set that variable:
- `POSTGRES_DB_PORT`: postgres port on your database server (defaults to `5432`)

### Linking to a postgres container
Linking to a postgres container, is pretty straightforward:
- `--link <some-postgres-container>:postgres`

For instance, if you want to run the official image for postgres, you could launch it like this:
```console
$ docker run --name some-postgres -p 5432:5432 -d postgres
```
And then you could launch geonetwork, linking to this container, and setting the required environment variables:
```console
$ docker run --name geonetwork -d -p 8080:8080 --link some-postgres:postgres -e POSTGRES_DB_USERNAME=postgres -e POSTGRES_DB_PASSWORD=mysecretpassword doublebyte/geonetwork-2:postgres
```
### Connecting to a postgres instance
If you want to connect to a postgres server running somewhere, you need to pass an extra environment variable, containing the IP address for this server (which could be localhost, if you are running it locally).
- `POSTGRES_DB_HOST`: IP address of your database server

For instance, if the server is running on `192.168.1.10`, on port `5434`, the username is `postgres` and the password is `mysecretpassword`:
```console
$ docker run --name geonetwork -d -p 8080:8080 -e POSTGRES_DB_HOST=192.168.1.10 -e POSTGRES_DB_PORT=5434 -e POSTGRES_DB_USERNAME=postgres -e POSTGRES_DB_PASSWORD=mysecretpassword doublebyte/geonetwork-2:postgres
```

# License

View [license information](http://www.geonetwork-opensource.org/manuals/trunk/eng/users/overview/license.html) for the software contained in this image.

# Supported Docker versions

This image is officially supported on Docker version 1.12.1.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.

# User Feedback

## Issues

If you have any problems with or questions about this image, please contact us through a [GitHub issue](https://github.com/GeoCat/geonetwork-2/issues). Please note that _these issues concern only the docker image, and not the geonetwork software itself_.

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/GeoCat/geonetwork-2/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.
