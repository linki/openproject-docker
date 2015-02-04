# OpenProject Docker

This repository intends to collect several ways of running openproject through docker.

We're currently running openproject through phusion's famous passenger baseimage, running
all sorts of helper processes inside the same container. It works but we find it quite unflexible.

A much more interesting approach is to run many single service containers independently and link them together.

So this repo's intention is to showcase many ideas and acts as a blueprint for real setups.

In addition, usage of orchestration tools is highly welcome (speak `fig.yml`, fleet unit files, etcd, consul, mesos...)

## Simple

Very basic openproject setup.

Consists of a `postgres`, `web` and `worker` container. No mounted volumes for persistence.

* starts with a stock ubuntu image
* adds various system dependencies through `apt-get`
* checks out openproject from source
* does bundler and npm magic
* runs the default rails server

## Development

A minimal setup that mounts your local checkout into the container in order to hack on it.

tbd.

## Passenger

A setup based on phusion's passenger baseimage

tbd.

## Packager

Use openproject's packager.io package inside a container.

tbd.

## Nginx, HAProxy

Show a simple static setup containing two app server containers behind a reverse proxy container.

tbd.

## Consul, etcd, registrator, skydns

Let's showcase some cool orchestration tools and allow spawning many app containers at any time,
registering them with consul et al. at startup and deregister on teardown.

tbd.

## CoreOS, fleet, etcd

Showcase some fleet unit files to run openproject on coreos.

tbd.

## Mesos / Marathon

That should be fun.

tbd.

## Raspberry PI / Resin.io

Bring openproject on a raspberry pi inside a container.

tbd.

## Kubernestes / Google Container Engine

Looking forward to this.

tbd.

## Amazon ECS

Hello World

tbd.

# Vulcand

HAProxy? Vulcand!

tbd.
