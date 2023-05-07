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

