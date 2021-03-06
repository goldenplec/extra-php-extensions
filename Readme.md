# Bref Extra PHP Extension

This repository has some AWS Lambda layers with PHP extensions. This is useful when you want something "off the shelf".
If you ever need more than 2-3 layer you should consider creating your own layer. That is because AWS has
a limit of 5 layers per Lambda.

We are happy to get contributions for other extensions. Sky is the limit! (And also your knowledge with Docker...)

## Install and configure

```cli
composer require bref/extra-php-extensions
```

```yaml
# serverless.yml
service: app

provider:
    name: aws
    region: us-east-1
    runtime: provided

plugins:
    - ./vendor/bref/bref
    - ./vendor/bref/extra-php-extensions # <----- Add the extra Serverless plugin

functions:
    console:
        handler: bin/console
        layers:
            - ${bref:layer.php-74}
            - ${bref:extra.amqp-php-74} # <----- Example for AMQP layer
            - ${bref:layer.console}
```

### Extra configuration files

You may add extra configuration files for PHP in the ``php/conf.d/`` directory.

```ini
;php/conf.d/amqp.ini
; Loading the AMQP extension.
extension=/opt/bref-extra/amqp.so
```

```ini
;php/conf.d/php.ini
memory_limit=128M
```

### Available layers

| Name       | Serverless config (php 7.4)      | php.ini config                             |
|:-----------|:---------------------------------|:-------------------------------------------|
| AMQP       | `${bref:extra.amqp-php-74}`      | `extension=/opt/bref-extra/amqp.so`        |
| Blackfire  | `${bref:extra.blackfire-php-74}` | `extension=/opt/bref-extra/blackfire.so`   |
| Cassandra  | `${bref:extra.cassandra-php-74}` | `extension=/opt/bref-extra/cassandra.so`   |
| DS         | `${bref:extra.ds-php-74}`        | `extension=/opt/bref-extra/ds.so`          |
| GD         | `${bref:extra.gd-php-74}`        | `extension=/opt/bref-extra/gd.so`          |
| GMP        | `${bref:extra.gmp-php-74}`       | `extension=/opt/bref-extra/gmp.so`         |
| gRPC       | `${bref:extra.grpc-php-74}`      | `extension=/opt/bref-extra/grpc.so`        |
| Igbinary   | `${bref:extra.igbinary-php-74}`  | `extension=/opt/bref-extra/igbinary.so`    |
| Imagick    | `${bref:extra.imagick-php-74}`   | `extension=/opt/bref-extra/imagick.so`     |
| LDAP       | `${bref:extra.ldap-php-74}`      | `extension=/opt/bref-extra/ldap.so`        |
| Mailparse  | `${bref:extra.mailparse-php-74}` | `extension=/opt/bref-extra/mailparse.so`   |
| Memcache   | `${bref:extra.memcached-php-74}` | `extension=/opt/bref-extra/memcache.so`    |
| Memcached  | `${bref:extra.memcached-php-74}` | `extension=/opt/bref-extra/memcached.so`   |
| MongoDB    | `${bref:extra.mongodb-php-74}`   | `extension=/opt/bref-extra/mongodb.so`     |
| MsgPack    | `${bref:extra.msgpack-php-74}`   | `extension=/opt/bref-extra/msgpack.so`     |
| ODBC Snowflake | `${bref:extra.odbc-snowflake-php-74}` | `extension=/opt/bref-extra/odbc.so` |
| Pcov       | `${bref:extra.pcov-php-74}`      | `extension=/opt/bref-extra/pcov.so`        |
| PostgreSQL | `${bref:extra.pgsql-php-74}`     | `extension=/opt/bref-extra/pgsql.so`       |
| Redis      | `${bref:extra.redis-php-74}`     | `extension=/opt/bref-extra/redis.so`       |
| Scrypt     | `${bref:extra.scrypt-php-74}`    | `extension=/opt/bref-extra/scrypt.so`      |
| SQLSRV     | `${bref:extra.sqlsrv-php-74}`    | `extension=/opt/bref-extra/sqlsrv.so`      |
| SQLSRV (PDO) | `${bref:extra.sqlsrv-php-74}`  | `extension=/opt/bref-extra/pdo_sqlsrv.so`  |
| UUID       | `${bref:extra.uuid-php-74}`      | `zend_extension=/opt/bref-extra/uuid.so`   |
| Xdebug     | `${bref:extra.xdebug-php-74}`    | `zend_extension=/opt/bref-extra/xdebug.so` |
| Yaml       | `${bref:extra.yaml-php-74}`      | `extension=/opt/bref-extra/yaml.so`        |

Note that the "Memcached" layer provides both extension for [Memcache](https://pecl.php.net/package/memcache) and [Memcached](https://pecl.php.net/package/memcached).

Note that you cannot use both the built-in imagick extension and imagick extension from this package.
This version of imagick is built against newer version of ImageMagick than the built-in one and provides heic and webp support.

### Blackfire installation

The Blackfire layer only have the probe installed.

You still need to install the agent.
The agent is installed on a separate server (not a lambda function). The micro
EC2 instance is sufficient to run the Blackfire agent.

Create a `blackfire.ini` file in `php/conf.d/` for your lambda function where you load the extension
and modify the `agent_socket` in order to point it to the Blackfire Agent.

```ini
;php/conf.d/blackfire.ini
extension=/opt/bref-extra/blackfire.so

blackfire.agent_socket = tcp://ip-172-40-40-40.eu-central-1.compute.internal:8307
blackfire.agent_timeout = 0.25
```

> You may tweak other Blackfire parameters.
> [See Blackfire documentation about them](https://blackfire.io/docs/configuration/php#configuring-the-probe-via-the-php-ini-configuration-file).

Then modify your [agent config](https://blackfire.io/docs/reference-guide/configuration#agent-configuration)
to make sure you are listening to `tcp://0.0.0.0:8307`.

This [blog post](https://developer.happyr.com/installing-blackfire-multiple-servers)
could be helpful as it describes how to install the Blackfire Agent.

### ODBC Snowflake setup

Enable the odbc extension in your `php.ini` by setting `extension=/opt/bref-extra/odbc.so`.
Then you also need to set the environment variable `ODBCSYSINI: /opt/snowflake_odbc/conf/` in your `serverless.yaml`
in order to tell unixODBC to load the ini file of the snowflake client.

You can then use snowflake like this: `odbc_connect('DRIVER=SnowflakeDSIIDriver;Server=[name].snowflakecomputing.com;Account=;Schema=;Warehouse=;Database=;Role=', 'username', 'password')`.

There is more information about the driver ini configuration in the [snowflake client documentation](https://docs.snowflake.com/en/user-guide/odbc-linux.html#step-4-configure-the-odbc-driver)
but the default configuration is enough in most cases.
The easiest way review those is to download the [`snowflake_odbc` directory](https://sfc-repo.snowflakecomputing.com/odbc/linux/index.html).

### SQL Server setup

The SQL Server layer includes both the SQLSRV extension and the PDO_SQLSRV extension ([source](https://github.com/microsoft/msphpsql)).
If you are unsure of which extension to use, this [stackoverflow post](https://stackoverflow.com/questions/11614536/sqlsrv-driver-vs-pdo-driver-for-php-with-ms-sql-server) may be helpful.
In addition to enabling the chosen extension in your `php.ini`, you must set the environment variable `ODBCSYSINI: /opt/microsoft/conf/`
in your `serverless.yaml` in order to tell unixODBC to load the required ini files.

## Docker images

There are Docker images for every layer. They are updated on every push to master
and on every tag. The name of the image is `bref/extra-[name]-php-[version]`. Find
all images on [Docker hub](https://hub.docker.com/u/bref).

These are the same docker images that creates the layers. All layer files lives inside
the `/opt` directory in the image.

## For contributors and maintainers

### Creating a new layer

The idea is to start from bref/build-php-XX, install all libraries and extensions
you want, then move all related files to `/opt`. Those files will be available in
the same same location on the Lambda.

Note that one can't just move files/libraries around. Most of them are expected to
be in their "standard" location.

1. Create a new folder in `layers` and name it to your extension name.
2. Add your Dockerfile
3. Update .travis.yml to include your extension
4. Update the table in the readme

Please refer [here](docs/create_your_own_extension_layer.md) for more details.

### Deploy new versions

#### Use Github actions

Prepare the changelog with some release notes. Then push your changes to `prepare-release` branch.
The Github Action will build an publish layers and then commit the `checksums.json`
and `layers.json` to your PR.

Now you will just merge and create a tag.

#### The manual way

```
export AWS_PROFILE=my_profile
make publish
git add checksums.json layers.json
git commit -m "New version of layers"
git push
```

#### Config

We build the docker layers in parallel if the GNU `parallel` command is available.
To make sure to build docker images in sequence run `make publish parallel=false`

You can also build only one specific layer by providing `layer=blackfire` to `make`.
Same thing for some specific version(s) of php by providing `php_versions="73 74"` to `make`.
You can invoke both ways:
```shell
# First way: make with named arguments
make layer=gd php_versions=74
# Second way: environment variables passed to make
layer=blackfire php_versions=80 make
```


