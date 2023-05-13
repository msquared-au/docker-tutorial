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

1.  Create a volume for the configuration: run `docker volume create proxy-conf`
1.  Run a temporary docker instance to access the volume: run `docker run -it --rm --volume=proxy-conf/mount/proxy-conf ubuntu bash`
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
        server_name app1.yourdomain.tld;
        location    / {
            proxy_pass http://app1;
        }
    }
    server {
        listen      80;
        server_name app2.yourdomain.tld;
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
1.  You should now be able to connect to app1 and app2 at http://app1.yourdomain.tld/ and http://app2.yourdomain.tld/ without using custom ports
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
* To reload nginx config: `nginx -s reload`
  * eg: to reload the proxy from the main host, run `docker exec proxy nginx -s reload`

## Replace our simple example nginx proxy with one that has some powerful automation built-in

1.  fetch the nginx-proxy docker image: docker pull nginxproxy/nginx-proxy
1.  in this order, run these commands:
    1.  `docker run -it --rm --name=app1 --net=proxy-net --volume=app1-web:/usr/share/nginx/html --env=VIRTUAL_HOST=app1.yourdomain.tld nginx`
    1.  `docker run -it --rm --name=proxy --net=proxy-net -p 80:80 --volume=/var/run/docker.sock:/tmp/docker.sock:ro nginxproxy/nginx-proxy`
    1.  `docker run -it --rm --name=app2 --net=proxy-net --volume=app2-web:/usr/share/nginx/html --env=VIRTUAL_HOST=app2.yourdomain.tld nginx`
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

### Troubleshooting information:

* Configuration file that's maintained by docker-gen: /etc/nginx/conf.d/default.conf
* More information:
  * https://github.com/nginx-proxy/nginx-proxy
  * https://hub.docker.com/r/nginxproxy/nginx-proxy

## Add SSL support via a separate container that integrates with the proxy container

1.  Fetch the Docker image: run `docker pull nginxproxy/acme-companion`
1.  Create the volumes needed for sharing certs and config between the proxy container
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
        --env=VIRTUAL_HOST=app1.yourdomain.tld
        --env=LETSENCRYPT_HOST=app1.yourdomain.tld nginx`
1.  You should be able to go to app1.yourdomain.tld and automatically be
    redirected to https://app1.yourdomain.tld - congratulations, you set up SSL
    with zero effort!
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
        docker run -it --rm --name=app2 --net=proxy-net
        --volume=app2-web:/usr/share/nginx/html
        --env=VIRTUAL_HOST=app2.yourdomain.tld
        --env=LETSENCRYPT_HOST=app2.yourdomain.tld
        nginx
1.  You should be able to visit that site, too, and be redirected to https!

### Additional information

* https://github.com/nginx-proxy/acme-companion

