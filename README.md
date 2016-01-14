# osixia/light-baseimage:0.2.1

[![](https://badge.imagelayers.io/osixia/light-baseimage:latest.svg)](https://imagelayers.io/?images=osixia/light-baseimage:latest 'Get your own badge on imagelayers.io') | Latest release: 0.2.1 -  [Changelog](CHANGELOG.md)
 | [Docker Hub](https://hub.docker.com/r/osixia/light-baseimage/) 

A Debian based docker image to help you build reliable image quickly. This image provide a simple opinionated solution to build multiple or single process image.

The aims of this image is to be used as a base for your own Docker images. It's base on the awesome work of: [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker)

Other base distribution are available:
 - [Debian Experimental](https://github.com/osixia/docker-light-baseimage/tree/experimental-light-baseimage) | [Docker Hub](https://hub.docker.com/r/osixia/experimental-light-baseimage/)
 - [Ubuntu 14:04](https://github.com/osixia/docker-light-baseimage/tree/ubuntu-light-baseimage) | [Docker Hub](https://hub.docker.com/r/osixia/ubuntu-light-baseimage/)

Table of Contents
- [Contributing](#contributing)
- [Overview](#overview)
- [Quick Start](#quick-start)
	- [Image directories structure](#image-directories-structure)
	- [Service directory structure](#service-directory-structure)
	- [Create a single process image](#create-a-single-process-image)
		- [Overview](#create-a-single-process-image)
		- [Dockerfile](#dockerfile)
		- [Service files](#service-files)
		- [Environment files](#environment-files)
			- [default.yaml](#defaultyaml)
			- [default.yaml.startup](#defaultyamlstartup)
		- [Build and test](#build-and-test)
	- [Create a multiple process image](#create-a-multiple-process-image)
		- [Overview](#create-a-multiple-process-image)
		- [Dockerfile](#dockerfile-1)
		- [Service files](#service-files-1)
		- [Build and test](#build-and-test-1)
- [Images based on light-baseimage](#images-based-on-light-baseimage)
- [Image Assets](#image-assets)
	- [Tools](#image-assets)
	- [Services available](#services-available)
- [Advanced User Guide](#advanced-user-guide)
	- [Service available](#service-available)
	- [Complex environment variables](#complex-environment-variables)
	- [Mastering image tools](#mastering-image-tools)
		- [run](#run)
			- [Run directory setup](#run-directory-setup)
			- [Startup files environment setup](#startup-files-environment-setup)
			- [Startup files execution](#startup-files-execution)
			- [ Process environment setup](#process-environment-setup)
			- [Process execution](#process-execution)
				- [Single process image](#single-process-image)
				- [Multiple process image](#multiple-process-image)
				- [No process image](#no-process-image)
			- [Extra environment variables](#extra-environment-variables)
		- [log-helper](#log-helper)
		- [complex-bash-env](#complex-bash-env)
	- [Tests](#tests)
- [Changelog](#changelog)

## Contributing

If you find this image useful here's how you can help:

- Send a pull request with your kickass new features and bug fixes
- Help new users with [issues](https://github.com/osixia/docker-openldap/issues) they may encounter
- Support the development of this image and star this repo !

## Overview

This image takes all the advantages of [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker) but makes programs optionals to allow more lightweight images and single process images. It also define simple directory structure and files to set quickly how a program (here called service) is installed, setup and run.

So major features are:
 - Simple way to install services and multiple process image stacks (runit, cron, syslog-ng-core and logrotate) if needed.
 - Getting environment variables from **.yaml** and **.json** files.
 - Special environment files **.yaml.startup** and **.json.startup** deleted after image startup files first execution to keep the image setup secret.
 - Greats build tools to minimize the image number of layers.

## Quick Start

### Image directories structure

This image use four directories:

- **/container/environment**: for environment files.
- **/container/service**: for services to install, setup and run.
- **/container/service-available**: for service that may be on demand downloaded, installed, setup and run.
- **/container/tool**: for image tools.

By the way at run time an other directory is create:
- **/container/run**: To store container run environment, state, startup files and process to run based on files in  /container/environment and /container/service directories.

But we will see that in details right after this quick start.

### Service directory structure

This section define a service directory that can be added in /container/service or /container/service-available.

- **my-service**: root directory
- **my-service/install.sh**: install script (not mandatory).
- **my-service/startup.sh**: startup script to setup the service when the container start (not mandatory).
- **my-service/process.sh**: process to run (not mandatory).
- **my-service/...** add whatever you need!

Ok that's pretty all to know to start building our first images!

### Create a single process image

#### Overview
For this example we are going to perform a basic nginx install.

See complete example in: [example/single-process-image](example/single-process-image)

First we create the directory structure of the image:

 - **single-process-image**: root directory
 - **single-process-image/service**: directory to store the nginx service.
 - **single-process-image/environment**: environment files directory.
 - **single-process-image/Dockerfile**: the Dockerfile to build this image.

**service** and **environment** directories name are arbitrary and can be changed but make sure to adapt their name in the everywhere and especially in the Dockerfile.

Let's now create the nginx service directory:

 - **single-process-image/service/nginx**: service root directory
 - **single-process-image/service/nginx/install.sh**: service installation script.
 - **single-process-image/service/nginx/startup.sh**:  startup script to setup the service when the container start.
 - **single-process-image/service/nginx/process.sh**: process to run.


#### Dockerfile

In the Dockerfile we are going to:
  - Download nginx from apt-get.
  - Add the service directory to the image.
  - Install service and clean up.
  - Add the environment directory to the image.
  - Define ports exposed and volumes if needed.

        # Use osixia/light-baseimage
        # https://github.com/osixia/docker-light-baseimage
        FROM osixia/light-baseimage:0.2.1-dev
        MAINTAINER Your Name <your@name.com>

        # Download nginx from apt-get
        RUN apt-get -y update \
            && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
               nginx

        # Add service directory to /container/service
        ADD service /container/service

        # Use baseimage install-service script and clean all
        # https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/install-service
        RUN /container/tool/install-service \
            && apt-get clean \
            && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

        # Add default env directory
        ADD environment /container/environment/99-default

        # Set /var/www/ in a data volume
        VOLUME /var/www/

        # Expose default http and https ports
        EXPOSE 80 443


The Dockerfile contains directives to download nginx from apt-get but all the initial setup will take place in install.sh file (called by /container/tool/install-service tool) for a better build experience. The time consumer download task is decoupled from the initial setup to make great use of docker build cache. If install.sh file is changed the builder won't have to download again nginx add will just run install scripts.

#### Service files

##### install.sh

This file must only contains directives for the service initial setup. If there is files to download, apt-get command to run we will it takes place in the Dockerfile for a better image building experience (see [Dockerfile](#Dockerfile)).

In this example, for the initial setup we just delete the default nginx debian index file and create a custom index.html:

    #!/bin/bash -e
    # this script is run during the image build

    rm -rf /var/www/html/index.nginx-debian.html
    echo "Hi!" > /var/www/html/index.html

Make sure install.sh can be executed (chmod +x install.sh).

Note: The install.sh script is run during the docker build so run time environment variables can't be used to customize the setup. This is done in the startup.sh file.


##### startup.sh

This file is used to make process.sh ready to be run and customize the service setup based on run time environment.

For example at run time we would like to introduce ourself so we will use an environment variable WHO_AM_I set by command line with --env. So we add WHO_AM_I value to index.html file but we want to do that only on the first container start because on restart the index.html file will already contains our name:

    #!/bin/bash -e
    FIRST_START_DONE="${CONTAINER_STATE_DIR}/nginx-first-start-done"

    # container first start
    if [ ! -e "$FIRST_START_DONE" ]; then
      echo "I'm ${WHO_AM_I}."  >> /var/www/html/index.html
      touch $FIRST_START_DONE
    fi

    exit 0

Make sure startup.sh can be executed (chmod +x startup.sh).

As you can see we use CONTAINER_STATE_DIR variable, it contains the directory where container state is saved, this variable is automatically set by run tool. Refer to the Advanced User Guide for more information.

##### process.sh

This file define the command to run:

    #!/bin/bash -e
    exec /usr/sbin/nginx -g "daemon off;"

Make sure process.sh can be executed (chmod +x process.sh).

*Caution: The command executed must start a foreground process otherwise the container will immediately stops.*

That why we run nginx with `-g "daemon off;"`

That's it we have a single process image that run nginx !
We could already build and test this image but 2 more minutes to take advantage of environment files !

#### Environment files

Let's create two files:
 - single-process-image/environment/default.yaml
 - single-process-image/environment/default.yaml.startup

##### default.yaml
default.yaml file define variables that can be used at anytime in the container environment:

    WHO_AM_I: We are Anonymous. We are Legion. We do not forgive. We do not forget. Expect us.

##### default.yaml.startup
default.yaml.startup define variables that are only available during the container **first start** in **startup files**.
\*.yaml.startup are deleted right after startup files are processed for the first time,
then all variables they contains will not be available in the container environment.

This helps to keep the container configuration secret. If you don't care all environment variables can be defined in **default.yaml** and everything will work fine.

But for this tutorial we will add a variable to this file:

    FIRST_START_SETUP_ONLY_SECRET: The bdd password is Baw0unga!

An try to get it's value in **startup.sh** script:

    #!/bin/bash -e
    FIRST_START_DONE="${CONTAINER_STATE_DIR}/nginx-first-start-done"

    # container first start
    if [ ! -e "$FIRST_START_DONE" ]; then
      echo ${WHO_AM_I}  >> /var/www/html/index.html
      touch $FIRST_START_DONE
    fi

    echo "The secret is: $FIRST_START_SETUP_ONLY_SECRET"

    exit 0

And in **process.sh** script:

    #!/bin/bash -e
    echo "The secret is: $FIRST_START_SETUP_ONLY_SECRET"
    exec /usr/sbin/nginx -g "daemon off;"

Ok it's time for the show !

#### Build and test

Build the image:

	docker build -t example/single-process --rm .

Start a new container:

    docker run -p 8080:80 example/single-process

Inspect the output and you should see that the secret is present in startup script:
> \*\*\* Running /container/run/startup/nginx...

> The secret is: The bdd password is Baw0unga!

And the secret is not defined in the process:
> \*\*\* Remove file /container/environment/99-default/default.yaml.startup [...]

> \*\*\* Running /container/run/process/nginx/run...

> The secret is:

Yes in this case it's not really useful to have a secret variable like this, but a concrete example can be found in [osixia/openldap](https://github.com/osixia/docker-openldap) image.
The admin password is available in clear text during the container first start to create a new ldap database where it is saved encrypted. After that the admin password is not available in clear text in the container environment.

Ok let's check our name now, go to [http://localhost:8080/](http://localhost:8080/)

You should see:
> Hi! We are Anonymous. We are Legion. We do not forgive. We do not forget. Expect us.

And finally, let's say who we really are, stop the previous container (ctrl+c) and start a new one:

    docker run --env WHO_AM_I="I'm Jon Snow, yes i'm not dead." \
    -p 8080:80 example/single-process

Refresh [http://localhost:8080/](http://localhost:8080/) and you should see:
> Hi! I'm Jon Snow, yes i'm not dead.

### Create a multiple process image

#### Overview

This example takes back the single process image example and add php5-fpm to run php scripts.

See complete example in: [example/multiple-process-image](example/multiple-process-image)

Note: it would have been  ♪ faster, better, stronger ♪ to extends the previous image but to make things easier we just copied files.

So here the image directory structure:

 - **multiple-process-image**: root directory
 - **multiple-process-image/service**: directory to store the nginx and php5-fpm service.
 - **multiple-process-image/environment**: environment files directory.
 - **multiple-process-image/Dockerfile**: the Dockerfile to build this image.

**service** and **environment** directories name are arbitrary and can be changed but make sure to adapt their name in the Dockerfile.

Let's now create the nginx and php5-fpm directories:

 - **multiple-process-image/service/nginx**: nginx root directory
 - **multiple-process-image/service/nginx/install.sh**: service installation script.
 - **multiple-process-image/service/nginx/startup.sh**:  startup script to setup the service when the container start.
 - **multiple-process-image/service/nginx/process.sh**: process to run.

 - **multiple-process-image/service/php5-fpm**: php5-fpm root directory
 - **multiple-process-image/service/php5-fpm/install.sh**: service installation script.
 - **multiple-process-image/service/php5-fpm/process.sh**: process to run.
 - **multiple-process-image/service/php5-fpm/config/default**: default nginx server config with

#### Dockerfile

In the Dockerfile we are going to:
  - Add the multple process stack
  - Download nginx and php5-fpm from apt-get.
  - Add the service directory to the image.
  - Install service and clean up.
  - Add the environment directory to the image.
  - Define ports exposed and volumes if needed.

        # Use osixia/light-baseimage
        # https://github.com/osixia/docker-light-baseimage
        FROM osixia/light-baseimage:0.2.1-dev
        MAINTAINER Your Name <your@name.com>

        # Add non free repository to apt-get (needed to download php5-fpm), install multiple process stack, nginx and php5-fpm
        # https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/add-multiple-process-stack
        RUN echo "deb http://http.debian.net/debian/ jessie main contrib non-free" >> /etc/apt/sources.list \
            && apt-get -y update \
            && /container/tool/add-multiple-process-stack \
            && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
               nginx \
               php5-fpm

        # Add service directory to /container/service
        ADD service /container/service

        # Use baseimage install-service script and clean all
        # https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/install-service
        RUN /container/tool/install-service \
            && apt-get clean \
            && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

        # Add default env directory
        ADD environment /container/environment/99-default

        # Set /var/www/ in a data volume
        VOLUME /var/www/

        # Expose default http and https ports
        EXPOSE 80 443


The Dockerfile contains directives to download nginx and php5-fpm from apt-get but all the initial setup will take place in install.sh file (called by /container/tool/install-service tool) for a better build experience. The time consumer download task is decoupled from the initial setup to make great use of docker build cache. If an install.sh file is changed the builder will not have to download again nginx and php5-fpm add will just run install scripts.

Maybe you already read that in the previous example ? Sorry.

#### Service files

Please refer to [single process image](#create-a-single-process-image) for the nginx service files description.
Here just php5-fpm files are described.

##### install.sh

This file must only contains directives for the service initial setup. If there is files to download, apt-get command to run we will it takes place in the Dockerfile for a better image building experience (see [Dockerfile](#Dockerfile) ).

In this example, for the initial setup we set some php5-fpm default configuration, replace the default nginx server config and add phpinfo.php file:

    #!/bin/bash -e
    # this script is run during the image build

    # config
    sed -i -e "s/expose_php = On/expose_php = Off/g" /etc/php5/fpm/php.ini
    sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" /etc/php5/fpm/php.ini
    sed -i -e "s/;listen.owner = www-data/listen.owner = www-data/g" /etc/php5/fpm/pool.d/www.conf
    sed -i -e "s/;listen.group = www-data/listen.group = www-data/g" /etc/php5/fpm/pool.d/www.conf

    # replace default website with php5-fpm default website
    cp -f /container/service/php5-fpm/config/default /etc/nginx/sites-available/default

    # create phpinfo.php
    echo "<?php phpinfo(); " > /var/www/html/phpinfo.php


Make sure install.sh can be executed (chmod +x install.sh).

##### process.sh

This file define the command to run:

    #!/bin/bash -e
    exec /usr/sbin/php5-fpm --nodaemonize

Make sure process.sh can be executed (chmod +x process.sh).

*Caution: The command executed must start a foreground process otherwise runit (use to supervise mutlple process images) will  keep restarting php5-fpm.*

That why we run php5-fpm with `--nodaemonize"`

##### config/default
nginx server configuration:

      server {
      	listen 80 default_server;
      	listen [::]:80 default_server;

      	root /var/www/html;

      	# Add index.php to the list if you are using PHP
      	index index.html index.htm index.nginx-debian.html;

      	server_name _;

      	location / {
      		# First attempt to serve request as file, then
      		# as directory, then fall back to displaying a 404.
      		try_files $uri $uri/ =404;
      	}

      	location ~ \.php$ {
      		fastcgi_split_path_info ^(.+\.php)(/.+)$;
      		# With php5-fpm:
      		fastcgi_pass unix:/var/run/php5-fpm.sock;
      		fastcgi_index index.php;
      		include fastcgi_params;
      		include fastcgi.conf;
      	}
      }

That's it we have a multiple process image that run nginx and php5-fpm !

#### Build and test


Build the image:

    docker build -t example/multiple-process --rm .

Start a new container:

    docker run -p 8080:80 example/multiple-process

Go to [http://localhost:8080/phpinfo.php](http://localhost:8080/phpinfo.php)

> phpinfo should be printed

So we have a container with two process supervised by runit running in our container !


## Images based on light-baseimage

Single process images:
- [osixia/openldap](https://github.com/osixia/docker-openldap)
- [osixia/openldap-backup](https://github.com/osixia/docker-openldap-backup)
- [osixia/keepalived](https://github.com/osixia/docker-keepalived)
- [osixia/tinc](https://github.com/osixia/docker-tinc)
- [osixia/registry-ldap-auth](https://github.com/osixia/docker-registry-ldap-auth)
- [osixia/cfssl-multirootca](https://github.com/osixia/docker-cfssl-multirootca)
- [osixia/backup-manager](https://github.com/osixia/docker-backup-manager)
- [osixia/mmc-agent](https://github.com/osixia/docker-mmc-agent)

Multiple process images:
- [osixia/mariadb](https://github.com/osixia/docker-mariadb)
- [osixia/wordpress](https://github.com/osixia/docker-wordpress)
- [osixia/roundcube](https://github.com/osixia/docker-roundcube)
- [osixia/phpMyAdmin](https://github.com/osixia/docker-phpMyAdmin)
- [osixia/phpLDAPadmin](https://github.com/osixia/docker-phpLDAPadmin)
- [osixia/kubernetes-reverseproxy](https://github.com/osixia/kubernetes-reverseproxy)
- [osixia/keepalived-confd](https://github.com/osixia/docker-keepalived-confd)
- [osixia/tinc-etcd](https://github.com/osixia/docker-tinc-etcd)
- [osixia/mmc-mail](https://github.com/osixia/docker-mmc-mail)
- [osixia/mmc-web](https://github.com/osixia/docker-mmc-web)

Send me a message to add your image based on light-baseimage in this list.

## Image Assets

### Tools

All container tools are available in `/container/tool` directory and are linked in `/sbin/` so they belong to the container PATH.


| Filename        | Description |
| ---------------- | ------------------- |
| run | The run tool is defined as the image ENTRYPOINT (see [Dockerfile](image/Dockerfile)). It set environment and run  startup scripts and images process. More information in the [Advanced User Guide / run](#run) section. |
| setuser | A tool for running a command as another user. Easier to use than su, has a smaller attack vector than sudo, and unlike chpst this tool sets $HOME correctly.|
| log-helper | A simple bash tool to print message base on the log level. |
|  add-service-available | A tool to download and add services in service-available directory to the regular service directory. |
| add-multiple-process-stack | A tool to add the multiple process stack: runit, cron syslog-ng-core and logrotate. |
| install-service | A tool that execute /container/service/install.sh and /container/service/\*/install.sh if file exists. |
|  complex-bash-env | A tool to iterate trough complex bash environment variables created by the run tool when a table or a list was set in environment files or in environment command line argument. |

### Services available

| Name        | Description |
| ---------------- | ------------------- |
| .runit | Replaces Debian's Upstart. Used for service supervision and management. Much easier to use than SysV init and supports restarting daemons when they crash. Much easier to use and more lightweight than Upstart. <br><br>*This service is part of the multiple-process-stack.*|
| .cron | Cron daemon. <br><br>*This service is part of the multiple-process-stack.*|
| .syslog-ng-core | Syslog daemon so that many services - including the kernel itself - can correctly log to /var/log/syslog. If no syslog daemon is running, a lot of important messages are silently swallowed. <br><br>Only listens locally. All syslog messages are forwarded to "docker logs".<br><br>*This service is part of the multiple-process-stack.* |
| .logrotate | Rotates and compresses logs on a regular basis. <br><br>*This service is part of the multiple-process-stack.*|
| .cfssl | CFSSL is CloudFlare's PKI/TLS swiss army knife. It's a command line tool for signing, verifying, and bundling TLS certificates. <br><br>Comes with cfssl-helper tool that make it docker friendly by taking command line parameters from environment variables. |


## Advanced User Guide

### Service available

A service-available is basically a normal service expect that it is in the `service-available` directory and have a `download.sh` file.

To add a service-available to the current image use the `add-service-available` tool. It will process the download.sh file of services given in argument and move them to the regular service directory (/container/service).

After that the service-available will be process like regular services.

Here simple Dockerfile example how to add a service-available to an image :

        # Use osixia/light-baseimage
        # https://github.com/osixia/docker-light-baseimage
        FROM osixia/light-baseimage:0.2.1-dev
        MAINTAINER Your Name <your@name.com>

        # Add cfssl tool, nginx and php5-fpm
        # https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/add-multiple-process-stack
        RUN apt-get -y update \
            && /container/tool/add-service-available .cfssl \
            && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
               nginx \
               php5-fpm
        ...


Note: Most of predefined service available start with a `.` to make sure they are install before regular services (so they can be used by regular services). The install-service tool process services in /container/service in alphabetical order.

To create a service-available just create a regular service, add a download.sh file to set how the needed content is download  and add it to /container/service-available directory. The download.sh script is not mandatory if nothing need to be downloaded.

For example a simple image example that add service-available to this baseimage: [osixia/web-baseimage](https://github.com/osixia/docker-web-baseimage)

### Complex environment variables
With light-baseimage you can set bash environment variable from .yaml and .json files.
But bash environment variables can't store complex objects such as table that can be defined in yaml or json file, that's why they are converted to complex bash environment variables (see [complex-bash-env](#complex-bash-env)  tool).

This complex environment variable in yaml:

    FRUITS:
      - orange
      - apple

Can also be set by command line converted in python or json:

    docker run -it --env FRUITS="#PYTHON2BASH:['orange','apple']" osixia/light-baseimage:0.2.1 printenv
    docker run -it --env FRUITS="#JSON2BASH:[\"orange\",\"apple\"]" osixia/light-baseimage:0.2.1 printenv


### Mastering image tools

#### run

The *Run tool* is defined as the image ENTRYPOINT (see [Dockerfile](image/Dockerfile)). It's the core tool of this image.

What it does:
- Setup the run directory
- Set the startup files environment
- Run startup files
- Set process environment
- Run process

*Run tool* takes several options, to list them:

    docker run osixia/light-baseimage:0.2.1 --help

##### Run directory setup
*Run tool* will create if they not exists the following directories:
  - /container/run/state
  - /container/run/environment
  - /container/run/startup
  - /container/run/process
  - /container/run/service

At the container first start it will search in /container/service or /container/run/service (if --copy-service option is used) all image's services.

In a service directory (e.g /container/service/my-service):
  - If a startup.sh file is found, the file is linked to /container/run/startup/my-service
  - If a process.sh file is found, the file is linked to /container/run/process/my-service/run

##### Startup files environment setup
*Run tool* takes all file in /container/environment/* and import the variables values to the container environment.
The container environment is then exported to /container/run/environment and in /container/run/environment.sh

##### Startup files execution
*Run tool* iterate trough /container/run/startup/ directory in alphabetical order and run the script.
At the end of the script execution the environment is restored to the version exported in /container/run/environment

Finally it try to run /container/run/startup.sh if exists.

##### Process environment setup
*Run tool* delete all .yaml.startup and .json.startup in /container/environment/* and clear the previous environment (/container/run/environment is removed)
Then it takes all remaining file in /container/environment/* and import the variables values to the container environment.
The container environment is then exported to /container/run/environment and in /container/run/environment.sh

##### Process execution

###### Single process image

*Run tool* execute the unique /container/run/process/service-name/run file.

If a main command is set for example:

    docker run -it osixia/openldap:1.1.0 bash

*Run tool* will execute the single process and the main command. If the main command exits the container exits. This is useful to debug or image development purpose.

###### Multiple process image

In a multiple process image the *run tool* execute runit witch supervise /container/run/process directory and start all services automatically. Runit will also relaunched them if they failed.

If a main command is set for example:

    docker run -it osixia/phpldapadmin:0.6.7 bash

*run tool* will execute runit and the main command. If the main command exits the container exits. This is still useful to debug or image development purpose.

###### No process image
If a main command is set *run tool* launch it otherwise bash is launched.
Example :

    docker run -it osixia/light-baseimage:0.2.1


##### Extra environment variables

*run tool* add 3 variables to the container environment:
- **CONTAINER_STATE_DIR**: /container/run/state
- **CONTAINER_SERVICE_DIR**: the container service directory. By default: /container/service but if the container is started with --copy-service option: /container/run/service
- **CONTAINER_LOG_LEVEL**: log level set by --loglevel option defaults to: 3 (info)

#### log-helper
This tool is a simple utility based on the CONTAINER_LOG_LEVEL variable to print leveled log messages.
For example if the log level is info:

    log-helper info hello

will echo:
> hello

    log-helper debug i'm bob

will not echo anything.

Note: log-helper support piped input

    echo "Heyyyyy" | log-helper info

> Heyyyyy

Available log message functions: error, warning, info, debug and trace.

Usage: log-helper error|warning|info|debug|trace message

You can also test the log level with the level function:

    log-helper level eq info && echo "log level is infos"

for example this will echo log level is trace if log level is trace.

Usage: log-helper level [eq|ne|gt|ge|lt|le](http://www.tldp.org/LDP/abs/html/comparison-ops.html) error|warning|info|debug|trace

#### complex-bash-env
With light-baseimage you can set bash environment variable from .yaml and .json files.
But bash environment variables can't store complex objects such as table that can be defined in json or yaml, that's why we need complex-bash-env tool.

For example the following yaml file:

      FRUITS:
        - orange
        - apple

will produce this bash environment variables:

      FRUITS=#COMPLEX_BASH_ENV:TABLE: FRUITS_ROW_1 FRUITS_ROW_2
      FRUITS_ROW_1=orange
      FRUITS_ROW_2=apple

(this is done by run tool)

complex-bash-env make it easy to iterate trough this variable:

      for fruit in $(complex-bash-env iterate "${FRUITS}")
      do
        echo $fruit
      done

More complete example can be found [osixia/phpLDAPadmin](https://github.com/osixia/docker-phpLDAPadmin) image.

### Tests

We use **Bats** (Bash Automated Testing System) to test this image:

> [https://github.com/sstephenson/bats](https://github.com/sstephenson/bats)

Install Bats, and in this project directory run:

	make test

## Changelog

Please refer to: [CHANGELOG.md](CHANGELOG.md)