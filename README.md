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
