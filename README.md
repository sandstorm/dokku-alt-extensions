# Sandstorm Media Extensions for [Dokku-Alt](https://github.com/dokku-alt/dokku-alt)

This package was originally forked from [dokku-nginx-vhosts-custom-configuration](https://github.com/neam/dokku-nginx-vhosts-custom-configuration);
but has been customized and modified heavily for our usage at Sandstorm Media.

## Installation

```bash
git clone https://github.com/sandstorm/dokku-alt-extensions.git /var/lib/dokku-alt/plugins/dokku-alt-extensions
```

## Features

- add custom configuration  to the nginx vhost configuration on the dokku host
- `volume:backup` backup volumes to tar files (everything inside `/app`)
- `volume:restore` restore volumes from tar files
- `fullbackup` backup the full dokku server. Currently supported:
  - global configuration
  - all volumes
  - MYSQL / MariaDB and Postgres databases
- `copy` copy a container; including its persistent volume and MariaDB database, to a new container. Easily replicate a setup for testing!
- `mariadb:sshtunnel` get the information on how to connect using Sequel Pro


## Creating a volume manually (e.g. replaying it from a backup)

If you dokku-ize an existing application, you need to put files etc at some point into a *volume*. For this, you
first need to prepare a special "tar" file, upload this one, and then run `dokku volume:restore` using this backup file.

In this process, make sure that all your persistent data resides in one or multiple folders underneath `/app`.

For the example, let's say you have persistent data inside `/app/Data/Persistent`. 

So, you create the structure in the following way:

```bash

# create base directory where we're working in
mkdir base-directory
cd base-directory

# this file contains information which apps are linked to it.
# We just need to create an empty file here.
touch .app-links

mkdir app

# now, create the directory you want to fill (which is persistent)
mkdir -p app/Data/Persistent

# here, place the files you need to have in that directory you just created.

# the ".paths" file *MUST* contain the directories which are persistent, so we
# add the "/app/Data/Persistent" directory there.
echo "/app/Data/Persistent" > .paths

# The tar command must look *exactly* like this in order to work
tar -cf ../volume.tar .

# Now, you can *validate* the tar-file using the following command:
tar -tf ../volume.tar
# In the above output, *all* files must start with "./" in order to work,
# and "./.paths" and "./.app-links" must appear in the list.
```

Now, upload the `volume.tar` file using SCP, optionally gzipping it before (in order to
reduce file size).

The following step must then be *executed on the dokku host itself*:

```bash
dokku volume:restore new_volume_name /uploaded/volume.tar

# Then, link the volume to the desired app:
dokku volume:link <app> new_volume_name
```


## Custom NGINX vhost Configuration

Relevant use cases for when the nginx vhost configuration needs to be customized can be to set proxy timeouts in order to allow long running requests, setting specific SSL directives, enabled uploading of large files and the like.


### Simple usage

1. Add a file containing your custom configuration to your app repo. For instance, you can call it `nginx.inc.conf` and commit it in the root of your app's repository.

2. Set the environment variable NGINX_VHOSTS_CUSTOM_CONFIGURATION to the in-container path to the file above:

```bash
$ dokku config:set <app> NGINX_VHOSTS_CUSTOM_CONFIGURATION=nginx.inc.conf             # Server side
$ ssh dokku@server config:set <app> NGINX_VHOSTS_CUSTOM_CONFIGURATION=nginx.inc.conf  # Client side
```

You're done! This plugin will read app environment variable with in-container path to custom configuration file and import the custom configuration to the app-specific configuration on the  dokku host.

### Additional commands

`nvcc:nginx.conf` will display the current nginx.conf

```bash
$ dokku nvcc:nginx.conf <app>             # Server side
$ ssh dokku@server nvcc:nginx.conf <app>  # Client side
```

`nvcc:nginx.conf.d` will display the current nginx.conf.d/ directory contents

```bash
$ dokku nvcc:nginx.conf.d <app>             # Server side
$ ssh dokku@server nvcc:nginx.conf.d <app>  # Client side
```

`nvcc:nginx-vhosts-custom-configuration.conf` will display the current nginx.conf.d/nginx-vhosts-custom-configuration.conf contents

```bash
$ dokku nvcc:nginx.conf.d <app>             # Server side
$ ssh dokku@server nvcc:nginx.conf.d <app>  # Client side
```

`nvcc:port` will display the current container port

```bash
$ dokku nvcc:port <app>             # Server side
$ ssh dokku@server nvcc:port <app>  # Client side
```

## License
MIT
