# microns-nda-access

This guide will walk you through setting up the database container and the access methods for the [microns_phase3_nda](https://github.com/cajal/microns_phase3_nda) data.

The data and files for these can be found in the microns-explorer [here](https://www.microns-explorer.org/cortical-mm3#f-data).

# Prerequisites

- ~115 GB of free disk space (around double that, ~220 GB, to load the image from the tar archive the first time)
- [Docker](https://docs.docker.com/desktop/)
- [docker-compose](https://docs.docker.com/compose/)

# Setup

This section comes in two parts, the first is the database containing the `microns-phase3-nda` schema and the second is for the container to access that data with DataJoint in a Jupyter notebook server that will come with tutorials or with the mysql-client.

If you know what you're doing and wish to handle importing the SQL file into an existing database yourself you can skip this next `Database` section and go straight to the `Access` section.

# Database

The docker image must first be downloaded from the microns-explorer (in a tar archive format, link is at the top).
Save this to an accessible location.

In the location where you've stored the downloaded image archive you then will load the image to your local filesystem:

```bash
docker load < functiona_data_database_container_image_v3.tar
```

OR

```bash
docker load --input functiona_data_database_container_image_v3.tar
```

To start the database you can either `Docker` or `docker-compose`:

The data is this database, started with both Docker or docker-compose, is not persistent and changes will be lost when exiting the container.

## docker-compose

This would be the preferred method for starting the service as it is more fully configured.
```bash
docker-compose up -d database
```

## Docker

Running the container without docker-compose is also an option.

```bash
docker run --network="host" --detach microns-phase3-nda-db-v3:latest
```

# Access

The data can be accessed in two ways, either with the mysql-client or through DataJoint in a Jupyter notebook service.

The default user and password for the database are:

`username:` root  
`password:` microns123

## Jupyter Notebook (DataJoint)

[//]: # (The pre-built image (microns-phase3-nda-notebook\) can be downloaded from the microns-explorer linked above.)

You can clone the access repository and build it yourself with `Docker` and `docker-compose`.
Clone the repository at https://github.com/cajal/microns_phase3_nda.

A `.env` (dotenv) file must be created in the same folder as `docker-compose.yml`, however it can be empty.

This can easily be done on linux with:

```bash
touch .env
```

but you can create the file however you wish.

Using the docker-compose you can start the service with:

```bash
docker-compose up -d notebook
```
which can then be accessed at http://localhost:8888/tree (or at a custom port set with env variable NOTEBOOK_PORT or set in an .env file in the same location as the docker-compose.yml).

http://localhost:8888 will send to Jupyter Lab, but not all plots/graphics will work out of the box without enabling jupyter lab extensions.

The database host will default to http://localhost:3306, but that points inside the container, in order to point to the outside mysql database you should either set an env variable of the name DJ_HOST (or in the .env file) or use the below Python snippet to set the host before loading the schema.

An external, persistent workspace can be mounted to the internal `workspace` folder by settings the `EXTERNAL_NOTEBOOKS` env variable to a folder of choice.

```python
import datajoint as dj

# This will be 127.0.1.1 if the container is on the same machine as the database, or just the hostname of the machine the database lives on.
dj.config['database.host'] = 'database-host-ip-goes-here'
dj.config['database.user'] = 'root'
df.config['database.password'] = 'microns123'

from phase3 import nda, func, utils
```

## mysql-client

From the local machine you can access it this way

```bash
mysql -h 127.0.1.1 -u root -p
```

which will then prompt for the password (the default from above is `microns123`) and will open an interactive mysql terminal.

The external hostname of the machine can be used in place of `127.0.1.1`.

## .env file

This docker-compose file is optimized to get a single machine up and running quickly with the database and notebook server.
However, you  might want to run a server and let many other clients connect to it, rather than having all the clients run their own database.

If so, you only need to run the notebook portion of the docker-compose file, but then you must modify the existing .env file to point to the host of an working database.  To do so you need to modify the DJ_HOST variable of the .env file provided.

replacing the "\<hostname>" with the hostname of the machine hosting the database (can use `127.0.1.1` if the notebook service has `network_mode: 'host'` enabled, but otherwise must use the network ip of the computer hosting the database container).

You can also replace the ./notebooks reference to a folder to a folder the of your choice.
