# CIG-SDK

## What is this for?

CIG-SDK is a helper script for all your container related development needs. It's for locally developing containerized applications.


## Install
___
### Install with Brew

Add custom Tap

```
$ brew tap itcig/tools
```

Install formula

```
$ brew install itcig/tools/cig-sdk
```

Run CLI tool for the first time and let it run through the installer

```
$ cig
```

Notes:
- When prompted for `BECOME password:`, enter your computer's admin password to allow Ansible to run commands as root. 
- During the generation of SSL certificates, your OS X Keychain will open a prompt asking you for your admin password again.

### Manual installation (WIP)

~~Install cig-sdk dependencies and start the development environment by running:~~

    $ curl -fsSL https://raw.githubusercontent.com/itcig/cig-sdk/master/setup/bootstrap | bash

~~If you're installing on a Ubuntu machine, run:~~

    $ curl -fsSL https://raw.githubusercontent.com/itcig/cig-sdk/master/setup/ubuntu | bash

~~When using linux and MacOS computers on a shared project and you have different docker-compose.yaml files for each, linux users need to add an environment variable for cig-sdk to be able to use the correct compose configuration. Add this to your chosen shell configuration (for example .bashrc or .zshrc).~~

    $ # Export compose file for ubuntu
    $ export COMPOSE_FILE="docker-compose-ubuntu.yaml"


## Update

Re-runs bootstrap installation script and reloads all service containers.

```
$ cig self update
```

## Usage

From a project directory that uses [docker-compose.yaml](https://docs.docker.com/compose/)

```
$ cig up
```


Quickly stop and start project containers. 

This is useful for refreshing the ENV or resetting the state of the container.

```
$ cig restart
```


Recreate all project containers and build fresh Docker images. 

This is slower than a restart but will give you a completely updated setup with the latest version of all remote Docker images.

```
$ cig reload
```

## What this tool includes
**cig-sdk** installs Docker plus addition applications and settings for a better development environment.
It sets up 4 services as Docker containers on your local system. 

### Service Containers

#### Nginx proxy
This project uses the nginx proxy container from [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy). This proxy reserves http (80) / https (443) ports on your localhost and proxies the requests to your project containers. Provide `VIRTUAL_HOST=your-address.test` env variable to your project and nginx proxy will take care of the rest.

#### Custom DNS server
We want to use real domain addresses for all containers. Some applications have strange behaviour if they are just used from `localhost:8080`. We use [andyshinn/dnsmasq](https://github.com/andyshinn/dnsmasq) for local dnsmasq which always responds `10.254.254.254` to any request. Installation script adds custom resolver file for your machine that matches any `.test` domain:

```
$ cat /etc/resolver/test
domain test
nameserver 10.254.254.254
search_order 1
```

This means that all `*.test` addresses are now pointing into your local machine so you don't have to edit your `/etc/hosts` file. We used `.test` tld because [it is reserved by IETF](https://en.wikipedia.org/wiki/.test) and will never be [sold to google](http://www.theregister.co.uk/2015/03/13/google_developer_gtld_domain_icann/) like what happened to it's popular cousin `.dev` or reserved names such as `.local`.

#### Custom HTTPS certificate generator
It's a really good practise to use https in production but only a few people use it in development. This makes it harder for people to notice `mixed content` error messages in development.

While using **cig-sdk** you won't see any of these:

![non-trusted https](https://cloud.githubusercontent.com/assets/5691777/13670188/1b042b48-e6d1-11e5-804e-542781b85ff5.png)

and instead more of these:

![self trusted https](https://cloud.githubusercontent.com/assets/5691777/13670189/1d697032-e6d1-11e5-99b5-aef757cb7f53.png)

**cig-sdk** includes custom certificate generator [onnimonni/signaler](https://github.com/onnimonni/signaler). **cig-sdk** installer creates a self-signed unique ca certificate during installation and saves it in your system keychain. If you provide `HTTPS_HOST=your-address.test` env variable to your project, a self-signed and trusted certificate will be created for your development environment.

_Note: If you load the site locally in your browser and see an insecure warning, try to wait and refresh the website or check the logs of the Signaler container by running._

```
$ cig service logs signaler
```

#### Custom SMTP server for debugging email
We included [mailhog/mailhog](https://hub.docker.com/r/mailhog/mailhog/) docker container for easier debugging of emails. Use `172.17.0.1:25` as SMTP server in your application and you are good to go.

Or if your legacy application has hard coded email server you can add the following to your `docker-compose.yaml`:

```
extra_hosts:
    - "my-mail-server.com:172.17.0.1"
```

and docker will add it into the `/etc/hosts` file inside the container automatically during startup.


## Short list of usual commands

```
# Reads docker-compose.yaml from current directory and starts up containers
$ cig up

# Stop and remove all containers in the project compose file
$ cig down

# Quickly pause the project and free resources
$ cig pause

# Wake the project up from pause as quickly as it was paused
$ cig unpause

# Open shell into any container (defaults to web service and bash)
$ cig shell

# Restart all project containers
$ cig reload

# List all containers from project
$ cig ps

# List all containers from docker
$ docker ps -a

# Open bash into any container
$ docker exec -it $CONTAINER_ID bash
```


## Workflow

- The source code running inside a project container is loaded from the directory on your hard drive. You can use text editors and Git clients on the host machines, and shouldn't need to work in the guest machine or the container.
- You should not need to run any application code directly from your host machine. Try to force yourself to find a containerized way of accomplishing things.
- Run `cig` without any arguments for lots of help

### Troubleshooting

#### No space left on device
Docker for Mac has only limited amount of disk space and this means that older images or stopped containers are taking all of the 60gb/120gb share.

To resolve this delete stopped containers, dangling images and dangling volumes. This can be done by running cleanup helper:

```
$ cig cleanup
```

If Docker for Mac still has a bug with freeing up disk space, dump databases you need and reset Docker for Mac settings. This will free all the space Docker is hogging. Then you will need to set up your projects again (import databases).

#### When in doubt, update and restart everything

To update all containers and settings run following global commands:
```
$ cig self update
$ cig reload
```


Copyright 2021 Capitol Information Group.
