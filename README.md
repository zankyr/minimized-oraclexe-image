# minimized-oraclexe-image

This project contains a minimized `Oracle 18.4.0 XE` Docker image intended to be used for integration testing.

This is a minimized image based on the official Oracle Docker XE Image but removes files not required for this purpose to minimize image file size and improve startup time.

## Image creation

First you need to create the official Oracle Docker image (this process can take several tens of minutes):

```
$ git clone https://github.com/oracle/docker-images.git
$ cd docker-images/OracleDatabase/SingleInstance/dockerfiles
$ ./buildContainerImage.sh -v 18.4.0 -x

$ docker images
REPOSITORY                         TAG         IMAGE ID       CREATED          SIZE
oracle/database                    18.4.0-xe   732e81a9b07f   28 minutes ago   5.89GB
```

Then use the Dockerfile from this project to build the minimized image:

```
$ git clone https://github.com/sky-uk/ita-sso-docker-images.git
$ cd ita-sso-docker-images/OracleDatabase/18.4.0-xe/
$ docker build . -t sso/oraclexe-light-image:18.4.0-xe

$ docker images
REPOSITORY                           TAG                   IMAGE ID       CREATED         SIZE
sso/oraclexe-light-image             18.4.0-xe             4e0e1d19134b   7 days ago      3.77GB
```

Pay attention to the image's dimensions: sometimes the build has errors, but the script doesn't fire any warning about.

## Usage
### Run the container

Once the image is built, you can run it as a basic container:

```bash
$ docker run --name local-oracle -d -p 1523:1521 sso/oraclexe-light-image:18.4.0-xe
```

### Connect to the containerized database

The username and password to connect to the database are: `SSO`.

The SID is: `XE`.

### Configure the database

Since the image is designed to be used in automation tests, it is provided with a working but empty Oracle instance, so
you need to configure the database once the container starts.

### Standalone instance

If you need to test something database related (procedures, DMLs, etc.) but you don't want to use a real environment,
you can start a container and then configure the database at your will. Simply, once the container started, connect to
it using your preferred client, using the information provided above.

![](doc/database_connection.png)

### Standalone instance with pre-configuration

The image is provided with a configuration script, `createUserAndTableSpace.sh`, which configures users and tablespace
when the container starts.

This script is currently deactivated. If you want to use it, uncomment the following `RUN instruction` inside
the `Dockerfile` and re-build the image:

```dockerfile
################################################################################################################
# execute user provided scripts.
################################################################################################################

# Uncomment this row if you want to configure the database (users, tablespaces, etc.) using the createUserAndTableSpace.sh
#RUN $ORACLE_HOME/createUserAndTableSpace.sh
```

### Run scripts at start time
If you want to execute scripts (bash or SQL) at startup time, you need to place your files inside
the `/docker-entrypoint-initdb.d`
container folder. All the files (again, only `.sh` or `.sql`) are executed by the `startup.sh` script, which is run via
the `CMD` instruction, thus it is executed at every start of the container.

# Contributions
A big thank-you goes to the creators of the image, https://github.com/diemobiliar/minimized-oraclexe-image
