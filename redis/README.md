# IBM Cloud Databases for Redis Migration Guide

This script will allow you to migrate your Compose for Redis database to IBM Cloud Databases for Redis. The script will copy the keys in your Compose for Redis database one-by-one to your Databases for Redis deployment. If you have time sensitive keys, those keys will be copied over as well with their TTL values. 

We recommend setting a maintenance window when you decide to migrate so that you stop writes to your Redis deployment. That way you don't need to run the script again to copy new keys, and you can then use your new database connection strings from Databases for Redis.

## Variables used in the migration script
You will need the following credentials when running the migration script:

- Compose hostname
- Compose port
- Compose database password
- IBM Cloud Databases for Redis hostname
- IBM Cloud Databases for Redis port
- IBM Cloud Databases for Redis password 
- Path to IBM Cloud Databases for Redis CA certificate

These can be gathered using the `ibmcloud cdb` command.

## Preparing your system

If you have a valid Python3 installation with the dependencies specified at the top of the script,
you can run the script directly (see next section). For a more isolated environment, or if your
system does not meet all the requirements, proceed to the following subsection.

### Install using Docker

This method is suitable for any operating system that supports Docker.
Make sure you have [docker](https://docs.docker.com/engine/install/) installed, then build an image as below.
Before doing that make sure you have downloaded the CA root certificate for the destination Redis service
and place it here with the name `ca.crt`. The build will fail if there is no such file under this name:

```bash
docker build . -t redis-migration:1
```

Once the above command succeeds, run the script using the following:

```bash
docker run -it --rm redis-migration:1 ...
```

Replace the `...` by the arguments you would normally provide as specified in the next section.
For example:

```bash
docker run -it --rm redis-migration:1 <source host> <source password> <source port> ... /home/ca.crt
```

Note from the above that the certificate for the destination was
already copied to `/home/ca.crt` from the build step.

## Running the script

Once you've got the credentials to your IBM Cloud Databases for Redis database, you can run the script using Python 3 from the terminal.

```shell
python3 redis_migration.py <source host> <source password> <source port> <destination host> <destination password> <destination port>  <destination ca certificate path> --sslsrc --ssldst
```

For example:

```shell
python3 redis_migration.py database.composedb.com mypassword123 99999 redis.test.databases.appdomain.cloud mypassword456 88888  ~/path/to/cert --sslsrc --ssldst
```

## Dry run

To test your migration, add the `--dryrun` flag. This is helpful for confirming connectivity, existing keys, etc.
without triggering the actual migration.

For example:

```shell
python3 redis_migration.py --dryrun ...
```

If destination flushing (`--flush`) is enabled in combination with `--dryrun`, the script will act as if the
destination were flushed without actually flushing it. The reported amount of already existing keys will always
be zero.
