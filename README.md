# docker-tutorial

Introduction to basic docker usage

## Introduction

This tutorial will demonstrate basic docker usage, including installation on an
empty virtual or physical host.  At the end of this tutorial, your host will be
serving up two different websites with SSL certificates via docker containers.

This tutorial will show you how to:
* Set up your host for docker
* Install and configure docker on your host
* Host websites in docker containers on your host
* Add SSL support for your websites
* Set up docker so that your website survive a reboot of your host

## Requirements

You will need:

* Basic familiarity with the Linux command-line
* A physical or virtual host that is accessible from the internet
  (this allows Let's Encrypt to see your sites and sign SSL
  certificates for them)
* A domain name (see below for how to do this without your own
  domain name)

For the purposes of this tutorial:

* `yourdomain.tld` means whatever domain you have; replace it
  with your domain name when entering commands
  * Note that host names that include `yourdomain.tld` should
    also have `yourdomain.tld` replaced with your domain name
  * Note that your domain name should not have
    [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)
    configured, or it will cause problems until you have SSL
    enabled on your site
* `mail@yourdomain.tld` means your email address; replace it with
  your email address when entering commands

## Set up your host

For this tutorial, we will use Ubuntu 22.04 LTS.  If a more-recent LTS version
is available, you can use that but be aware that some steps may vary slightly.

1.  Install Ubuntu 22.04 LTS
1.  Set up the time zone for the server:
    1.  Either run `tzselect` to browse timezones or `timedatectl list-timezones` to list timezones
        * You can use `grep` to search for your timezone. eg:
          `timedatectl list-timezones | grep -i toronto` to find the timezone for Toronto
    1.  Run `timedatectl set-timezone <timezone>` to set the timezone, replacing <timezone> with the timezone you found above
        * eg: `timedatectl set-timezone America/Toronto` to set the server's timezone to Toronto's timezone
1.  Update Ubuntu:
    1.  Run `apt-get update`
    1.  Run `apt-get upgrade`
        * There's no need to restart any services if it offers, since we'll restart the host at the end of this setup process anyway
1.  Set the hostname in `/etc/hostname` to `dockertest.yourdomain.tld`
1.  If you have set up a physical host and need to add remote access via SSH, run these commands:
    1.  `apt-get install openssh-server`
    1.  `sudo systemctl enable ssh --now`
1.  Reboot your host

## DNS setup

### For your own domain name

If you have your own domain name, add the following records:

* A hostname 'A' record `dockertest.yourdomain.tld` that points to your server's public IP address
* A wildcard 'C' record `*.dockertest.yourdomain.tld` that points to `dockertest.yourdomain.tld`

This will allow you to use arbitrary hostnames and have them all resolve to your host without
having to set each of them up manually.  For example:

 * `app1.dockertest.yourdomain.tld`
 * `app2.dockertest.yourdomain.tld`

### If you don't have a domain name

If you don't have a domain name of your own, you can use one of a handful of
DNS services that automatically provide arbitrary lookup for any IP address.
One such service is [nip.io](https://nip.io).

Once you have set up your host, you can embed its public IP address into a domain name based
on nip.io and and subdomains will automatically resolve to that IP address.

For example, if your host's public IP address were `10.20.30.40`, you could put
`dockertest.10.20.30.40.nip.io` into your web browser to connect to your host's
web service.  This service allows you to add arbitrary subdomains in the same
way that a wildcard DNS record lets you do this (as described above).  So, you
could also visit `app1.dockertest.10.20.30.40.nip.io` in your web browser and
visit your host's web service.

So, to use nip.io as if it were your own domain name, replace `yourdomain.tld`
with `10.20.30.40.nip.io` in the rest of this documentation (be sure to replace
`10.20.30.40` with your host's public IP address, of course).

## Install Docker Engine

You can install Docker Engine (aka "just Docker") from Ubuntu's repository, or
you can install the latest from the Docker project directly by following these
instructions:

1.  Follow these instructions: <https://docs.docker.com/engine/install/ubuntu/>
    * Consider using the steps for "Install using the apt repository"
    * These instructions include a step that checks your Docker installation
1.  If you want to be able to run Docker as a normal user (instead of as root),
    also follow these instructions (but pay attention to the security warnings
    before doing so): <https://docs.docker.com/engine/install/linux-postinstall/>

## Obtain the [nginx](https://www.nginx.com/) [Docker image](https://hub.docker.com/_/nginx)

In Docker, images contain your (or somoene else's) application and containers
are the environment in which that application is run.  Think of images as the
.exe file or the computer's filesystem, and the container as the memory and CPU
where the program is loaded and run when you want to use it.

1.  Download the nginx image: run `docker pull nginx`

## Test the nginx image

1.  Create and run a Docker container based on the nginx image:
    `docker run -it -p 80:80 --rm --name testnginx nginx`
1.  To confirm that the container is serving a website, run `docker ps`
1.  You should now be able to open a web browser and visit nginx's
    test page either via your host's IP address or via
    `dockertest.yourdomain.tld`
1.  You can stop nginx by pressing `^C` in the terminal where nginx
    is running ("`^C`" means "hold the `CTRL` key while pressing the `C` key)

### Troubleshooting information

* To connect to the container and explore it while it's running:
  `docker exec -it testnginx bash`
* The main nginx configuration file in the container is `/etc/nginx/nginx.conf`
* The webserver configuration file is `/etc/nginx/conf.d/default.conf`
* The folder containing the HTML files is `/usr/share/nginx/html/`

## Create backing stores for web files

In Docker, changes made to files inside a container are normally lost when the
container is stopped or restarted; this means that fixing a misbehaving
container can be as simple as restarting it.  However, it means that a separate
mechanism is needed for storing data.  Docker provides "volumes" for this
purpose.  Volumes can store data that you want to survive past a container, or
to share between containers.

So that we can store HTML files for our two demonstration sites, we will create
two Docker volumes:

1.  Run `docker volume create --name app1-web`
1.  Run `docker volume create --name app2-web`
1.  To see all volumes, run `docker volume ls`

## Test that the app1 web folder is accessible and persistent

1.  Launch an nginx container serving the files for app1:
    run `docker run -it -p 80:80 --rm --name app1 --volume=app1-web:/usr/share/nginx/html nginx`
1.  Confirm you can connect from a web browser and that you see the nginx welcome page
1.  Connect to the nginx container and make changes to the HTML pages:
    1.  Run `docker run -it --rm --volume=app1-web:/mnt/app1-web ubuntu bash`
    1.  Install vim: run `apt-get update ; apt-get install vim`
    1.  Edit the welcome page and change it: run `vim /mnt/app1-web/index.html`
1.  Confirm that you can still connect from a browser and that the welcome page has changed
1.  Shut down nginx with `^C`, then restart it with the command above
1.  Check that the changes are still present

You can perform the same set of steps but replace `app1` with `app2` to check
that the app2 web folder is accessible and persistent.

## Run and test two applications (containers) at the same time

We'll run two web servers at the same time, each in its own container.  So that
the containers don't get in one another's way, we'll map each one to a
different port on your host; you'll be able to connect to each one separately
by specifying different port numbers in the URL to supply to your web browser.

1.  To launch app1: run `docker run -it --rm -p 1080:80 --name=app1 --volume=app1-web:/usr/share/nginx/html nginx`
1.  To launch app2: run `docker run -it --rm -p 2080:80 --name=app2 --volume=app2-web:/usr/share/nginx/html nginx`
1.  To see app1: open a web browser window or tab for `http://dockertest.yourdomain.tld:1080`
1.  To see app2: open a web browser window or tab for `http://dockertest.yourdomain.tld:2080`
1.  Shut down each nginx container with `^C`

## Create a network for containers to share

To make it easy for the proxy container to connect to the other containers,
we create a Docker-internal network for the containers to share:

1.  Run `docker network create proxy-net`

## Create an instance of nginx to handle incoming connections

Creating a special-purpose "proxy" container gives us some useful capabilities:
* All apps can share the same ports (port 80 for http connections, and port 443 for https connections)
* https can be added to all apps without having to implement https support to any of the apps

1.  Create a volume for the configuration: run `docker volume create --name proxy-conf`
1.  Run a temporary docker instance to access the volume: run `docker run -it --rm --volume=proxy-conf:/mnt/proxy-conf ubuntu bash`
1.  Install vim: run `apt-get update ; apt-get install vim`
1.  Create `proxy.conf` in `/mnt/proxy-conf` and give it this content:
    ```nginx
    server {                # first server config is default when no other match
        listen      80;
        server_name _;      # don't match any legit host by accident
        return      503;    # return "service unavailable" http status
    }
    server {
        listen      80;
        server_name app1.dockertest.yourdomain.tld;
        location    / {
            proxy_pass http://app1;
        }
    }
    server {
        listen      80;
        server_name app2.dockertest.yourdomain.tld;
        location    / {
            proxy_pass http://app2;
        }
    }
    ```

## Run all three containers simultaneously and test

1.  In this order, run these three commands:
    1.  `docker run -it --rm --name=app1 --net=proxy-net --volume=app1-web:/usr/share/nginx/html nginx`
    1.  `docker run -it --rm --name=app2 --net=proxy-net --volume=app2-web:/usr/share/nginx/html nginx`
    1.  `docker run -it --rm --name=proxy --net=proxy-net -p 80:80 --volume=proxy-conf:/etc/nginx/conf.d nginx`
1.  You should now be able to connect to app1 and app2 at http://app1.dockertest.yourdomain.tld/ and http://app2.dockertest.yourdomain.tld/ without using custom ports
    * When you access app1, you should see activity in the proxy container and the app1 container
    * When you access app2, you should see activity in the proxy container and the app2 container
1.  When you're finished testing, shut down each nginx container with `^C`

### Notes

* There is no need to expose each app (note the lack of "-p" option when running the apps; only the proxy needs a "-p" option)
* The proxy instance connects to the app instances via the proxy network we set up earlier
* The proxy instance can see the other instances and connect to them via their names
* The proxy instance's configuration file uses the names of the other instances to connect to them

### Troubleshooting information

* nginx's configuration is at /etc/nginx
* To reload nginx configuration: `nginx -s reload`
  * eg: to reload the proxy from the main host, run `docker exec proxy nginx -s reload`

## Replace our simple example nginx proxy with one that has some powerful automation built-in

1.  fetch the nginx-proxy docker image: run `docker pull nginxproxy/nginx-proxy`
1.  In this order, run these commands:
    1.  `docker run -it --rm --name=app1 --net=proxy-net --volume=app1-web:/usr/share/nginx/html --env=VIRTUAL_HOST=app1.dockertest.yourdomain.tld nginx`
    1.  `docker run -it --rm --name=proxy --net=proxy-net -p 80:80 --volume=/var/run/docker.sock:/tmp/docker.sock:ro --env=TRUST_DOWNSTREAM_PROXY=false nginxproxy/nginx-proxy`
    1.  `docker run -it --rm --name=app2 --net=proxy-net --volume=app2-web:/usr/share/nginx/html --env=VIRTUAL_HOST=app2.dockertest.yourdomain.tld nginx`
1.  You should be able to access both app1 and app2 hosts via their respective
    hostnames, even though one was created before and one was created after the
    proxy was started up!
1.  When you're finished testing, shut down each nginx container with `^C`

### Notes

* There is no explicit configuration of the proxy instance here: the proxy instance finds
  the applications via the environment variable "VIRTUAL_HOST" specified on each instance
* There is a tool running in the proxy instance called "docker-gen" that monitors
  Docker for notifications of start-up and shut-down of other docker containers
* docker-gen uses the information to update the configuration file inside the
  proxy container and notify nginx that it needs to update its configuration
* When the proxy container starts up, docker-gen checks for any existing
  containers with the "VIRTUAL_HOST" environment variable, since otherwise it
  will have missed the start-up notification for containers that were launched
  before it
* The option `--env=TRUST_DOWNSTREAM_PROXY=false` is a security feature:
  * Normally, nginx instances are capable of making use of certain
    proxy-related headers in order to communicate with one another to improve
    performance and security
  * However, nginx instances that receive direct connections from the outside
    world should ignore these headers, as they could be forged in order to
    attempt to bypass security features
  * The nginx Docker image is configured to make use of these headers, to
    simplify the task of configuring multiple instances that collaborate,
    like ours are collaborating between the proxy instance and the application
    instances.
  * However, we should override that default on any public-facing instances
    to prevent a malicious actor from trying to trick our proxy into behaving
    in a way that is less secure.

### Troubleshooting information:

* Configuration file that's maintained by docker-gen: /etc/nginx/conf.d/default.conf
* More information:
  * https://github.com/nginx-proxy/nginx-proxy
  * https://hub.docker.com/r/nginxproxy/nginx-proxy

## Add SSL support via a separate container that integrates with the proxy container

1.  Fetch the Docker image: run `docker pull nginxproxy/acme-companion`
1.  Create the volumes needed for sharing certificates and configuration between the proxy container
    and the acme container
    (acme is the certificate protocol used by Let's Encrypt):
    1.  Run `docker volume create --name certs`
    1.  Run `docker volume create --name vhost`
    1.  Run `docker volume create --name html`
    1.  Run `docker volume create --name acme`
1.  Start the proxy container and the acme certificate companion container;
    in this order, run these commands:
    1.  `docker run -it --rm --name=proxy --net=proxy-net -p 80:80 -p 443:443
        --volume certs:/etc/nginx/certs --volume vhost:/etc/nginx/vhost.d
        --volume html:/usr/share/nginx/html
        --volume=/var/run/docker.sock:/tmp/docker.sock:ro
        --env=TRUST_DOWNSTREAM_PROXY=false
        nginxproxy/nginx-proxy`
    1.  `docker run -it --rm --name acme
        --volumes-from proxy
        --volume /var/run/docker.sock:/var/run/docker.sock:ro
        --volume acme:/etc/acme.sh
        --env "DEFAULT_EMAIL=mail@yourdomain.tld"
        nginxproxy/acme-companion`
1.  Start the first app container:
    1.  Run `docker run -it --rm --name=app1 --net=proxy-net
        --volume=app1-web:/usr/share/nginx/html
        --env=VIRTUAL_HOST=app1.dockertest.yourdomain.tld
        --env=LETSENCRYPT_HOST=app1.dockertest.yourdomain.tld nginx`
1.  You should be able to go to app1.dockertest.yourdomain.tld and automatically be
    redirected to https://app1.dockertest.yourdomain.tld - congratulations, you set up SSL
    with zero effort!
    * If you are not redirected to the https site, check the logs and wait
      to see if anything is happening; the acme companion might be in the
      process of generating certificates and reloading the proxy's
      configuration.  After the logs settle down, reload the app1
      page and see if it's redirected to the https version of the app1
      page.
1.  If you watch the proxy log, you'll very quickly see a handful of things
    connect to your new site, even though you might not yet have alerted anyone
    to its existence!
1.  Relax, this is part of Certificate Transparency: https://certificate.transparency.dev/
1.  Some sites that might monitor the Certificate Transparency logs and inspect
    your shiny new site include:
    * https://leakix.net/
    * https://domainsproject.org/
    * Google
1.  Now start the second app container:
        Run `docker run -it --rm --name=app2 --net=proxy-net
        --volume=app2-web:/usr/share/nginx/html
        --env=VIRTUAL_HOST=app2.dockertest.yourdomain.tld
        --env=LETSENCRYPT_HOST=app2.dockertest.yourdomain.tld
        nginx`
1.  You should be able to visit that site, too, and be redirected to https!
1.  Shut down each container with `^C`

### Additional information

* https://github.com/nginx-proxy/acme-companion

## Create containers that are persistent

While experimenting, we've been creating temporary containers that disappear
when we cancel them.  Now that we've finished experimenting, we want to create
containers that will run in the background and will restart when the host
restarts.

(Note: below we'll show an alternative mechanism for achieving this.)

1.  to create the containers run these commands in any order:
    * `docker container create --name=proxy --restart=unless-stopped --net=proxy-net
      -p 80:80 -p 443:443
      --volume certs:/etc/nginx/certs
      --volume vhost:/etc/nginx/vhost.d
      --volume html:/usr/share/nginx/html
      --volume=/var/run/docker.sock:/tmp/docker.sock:ro
      --env=TRUST_DOWNSTREAM_PROXY=false
      nginxproxy/nginx-proxy`
    * `docker container create --name acme --restart=unless-stopped
      --volumes-from proxy
      --volume /var/run/docker.sock:/var/run/docker.sock:ro
      --volume acme:/etc/acme.sh
      --env "DEFAULT_EMAIL=mail@yourdomain.tld"
      nginxproxy/acme-companion`
    * `docker container create --name=app1 --restart=unless-stopped --net=proxy-net
      --volume=app1-web:/usr/share/nginx/html
      --env=VIRTUAL_HOST=app1.dockertest.yourdomain.tld
      --env=LETSENCRYPT_HOST=app1.dockertest.yourdomain.tld
      nginx`
    * `docker container create --name=app2 --restart=unless-stopped --net=proxy-net
      --volume=app2-web:/usr/share/nginx/html
      --env=VIRTUAL_HOST=app2.dockertest.yourdomain.tld
      --env=LETSENCRYPT_HOST=app2.dockertest.yourdomain.tld
      nginx`
1.  Start the containers: run `docker start proxy acme app1 app2`
1.  Test that you can connect to both applications, that the applications are
    separate, and that you are redirected to https for the applications
1.  Reboot the host
1.  Test again!

If everything stayed up after a reboot, then you are done!

Congratulations, you went through the journey of setting up Docker containers
to run your applications, and hopefully learned a lot about Docker and about
hosting applications under Docker.

### Tips

* If you want to watch the logs for one of the containers while
  accessing your applications, run `docker logs --follow`.  For example,
  to watch the proxy logs run `docker logs proxy --follow`.

## Use "docker compose" to manage persisten containers (optional)

This step is optional, and is an alternative approach to the previous step for
creating persistent containers.

The way we created containers above is a procedural approach to creating
containers, that is each step creates, runs, or otherwise manages a single
container at a time.  "docker compose" provides a declarative way to manage a
set of containers, including setup of Docker volumes and Docker networks; the
set of containers can then be managed all at once with single commands.

1.  Shut down the containers we created above: run `docker stop app1 app2 proxy acme`
    * Tip: if you don't have any other containers running, this command will stop all containers: `docker stop $(docker ps -a -q)`
1.  Remove the containers we created above: run `docker rm app1 app2 proxy acme`
    * Tip: if you don't have any other containers on the system, this command will remove all stopped containers: `docker container prune`
1.  Create a folder called `web`, and within it create a file called `compose.yaml` with the following content:
    ```nginxproxy
    services:
      proxy:
        image: nginxproxy/nginx-proxy
        ports:
          - "80:80"
          - "443:443"
        networks:
          - proxy-net
        volumes:
          - certs:/etc/nginx/certs
          - vhost:/etc/nginx/vhost.d
          - html:/usr/share/nginx/html
          - /var/run/docker.sock:/tmp/docker.sock:ro
        environment:
          - TRUST_DOWNSTREAM_PROXY=false
      acme:
        image: nginxproxy/acme-companion
        networks:
          - proxy-net
        volumes_from:
          - proxy
        volumes:
          - acme:/etc/acme.sh
          - /var/run/docker.sock:/var/run/docker.sock:ro
        environment:
          - DEFAULT_EMAIL=mail@yourdomain.tld
      app1:
        image: nginx
        networks:
          - proxy-net
        volumes:
          - app1-web:/usr/share/nginx/html
        environment:
          - VIRTUAL_HOST=app1.dockertest.yourdomain.tld
          - LETSENCRYPT_HOST=app1.dockertest.yourdomain.tld
      app2:
        image: nginx
        networks:
          - proxy-net
        volumes:
          - app2-web:/usr/share/nginx/html
        environment:
          - VIRTUAL_HOST=app2.dockertest.yourdomain.tld
          - LETSENCRYPT_HOST=app2.dockertest.yourdomain.tld
    networks:
      proxy-net:
    volumes:
      acme:
      certs:
      vhost:
      html:
      app1-web:
      app2-web:
    ```
1.  Shut down the containers we created above: run `docker stop app1 app2 proxy acme`
    * Tip: this command will stop all containers: `docker stop $(docker ps -a -q)`
1.  Remove the containers we created above: run `docker rm app1 app2 proxy acme`
    * Tip: this command will remove all stopped containers: `docker container prune`
1.  Start the containers: `docker compose up`
1.  This will set everything up and run all the containers; it may take a while for everything to be ready
1.  Once the logging settles down, go to your web sites and check!  Note that the two web app contents will have been reset (see below for why, and how to fix this)
1.  Shut down the set of containers with `^C`
1.  Create the containers: `docker compose create` in the folder containing `compose.yaml`
1.  Start the containers in the backgound (so that `^C` doesn't stop them): run `docker compose start` in the folder containing `compose.yaml`
1.  Check that the websites are up again
1.  Reboot the host and check again to see if the applications came back up

### Notes

* `docker compose` creates namespaces for things based on the name of the
  folder containing the file `compose.yaml`:
  * For example, the web app volumes will be called `web_app1-web` and
    `web_app2-web` because we put `compose.yaml` in the folder `web`
  * The shared network will be named `web_proxy-net` for the same reason
* There's no way to rename volumes; if you want to use the existing data from
  the volumes you created before, you'll have to copy them yourself
  * For example: run
    `docker run --rm --volume=app1-web:/mnt/src --volume=web_app1-web:/mnt/dest ubuntu bash -c 'cp -a /mnt/src/* /mnt/dest'`
    to copy the contents of volume `app1-web` to the volume `web_app1-web`
  * Note: running the `cp` (copy) command via `bash` (a commonly used shell)
    allows bash to expand `*` to the list of files and folders in the volume,
   simplifying the process of copying everything between the volumes
* `docker compose` is intended to manage large deployments and perform scaling
  using multiple container instances; this means:
  * The project-name prefix mechanism might be confusing or get in the way when
    setting up a single set of containers
  * When setting up multiple sets of containers, consider creating a single
    folder to hold folders for your `compose.yaml` files: this will prevent
    two folders with `compose.yaml` from having the same name and thus
    from causing conflicts
  * Adding a new web host means adding a new container via `compose.yaml` then
    re-creating the containers via `docker compose down`,
    `docker compose up --detach` (the option `--detach` causes the containers
    to run in the background, like `docker compose start` does)
  * `docker compose` commands act on the file `compose.yaml` in the current
    folder

