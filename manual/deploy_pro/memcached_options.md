# Setup Memcached for Pro Edition

In Seafile Pro edition, a few advanced features rely on memcached.

* When clustering is enabled, some information is stored in memcached to allow access from multiple nodes. It includes copy/move progress, zip progress.
* Some information is cached in memcached to improve performance, such as library list, small objects from object storage backends, mapping from library ID to storage backends.

## Install Memcached

First, make sure `libmemcached` library and development headers are installed on your system.

```
# on Debian/Ubuntu 18.04+
apt-get install memcached libmemcached-dev -y
pip3 install --timeout=3600 pylibmc django-pylibmc

systemctl enable --now memcached

```

```
# on CentOS 8
yum install memcached libmemcached -y
pip3 install --timeout=3600 pylibmc django-pylibmc﻿

systemctl enable --now memcached

```

## Modify seahub_settings.py

Add the following configuration to `seahub_settings.py`.

```
CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
    },
}

```

## Modify seafile.conf

Since Seafile-pro-10.0.0, `[memcached]` option group must be configured in seafile.conf. You only need to set this one option to replace all previous memcached related options in seafile.conf.

```
[memcached]
memcached_options = --SERVER=<the IP of Memcached Server> --POOL-MIN=10 --POOL-MAX=100
```

Since Seafile-pro-11.0.0, redis cache is supported and you can use redis instead of memcached.

You can use redis cache by adding the following configuration:

```
[redis]
# the ip of redis server
redis_server = 127.0.0.1
# the port of redis server
redis_port = 6379
# the expire time of redis, the unit is second, default to 24h
redis_expriy = 86400
# the max number of connections to redis server, default to 100
max_connections = 100
```

If you configure `[redis]` and `[memcache]` option group at the same time, then redis will be used.

### For version 7.1 or before

The memcached configurations for different features are scattered in different sections in seafile.conf file. We list these options here for quick reference.

For [clustering](./deploy_in_a_cluster.md), the following options are provided.

```
[cluster]
enabled = true
memcached_options = --SERVER=<the IP of Memcached Server> --POOL-MIN=10 --POOL-MAX=100
```

For object storage backends, memcached options can be added in the backend configurations. You can refer to the corresponding documentation for specific backend. For example, when you use S3 backend,

```
[block_backend]
name = s3
# bucket name can only use lowercase characters, numbers, periods and dashes. Period cannot be used in Frankfurt region.
bucket = my-block-objects
key_id = your-key-id
key = your-secret-key
memcached_options = --SERVER=<the IP of Memcached Server> --POOL-MIN=10 --POOL-MAX=100
```

When you enable [multiple storage backend](./multiple_storage_backends.md), you can set the below options to cache mappings from library ID to its backend. This option also enables library list cache feature, which reduces database loads for frequently listing accessible library list.

```
[memcached]
memcached_options = --SERVER=<the IP of Memcached Server> --POOL-MIN=10 --POOL-MAX=100
```

