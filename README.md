# dokku graphite (beta) 

Grafana/graphite/statsd plugin for dokku. Currently defaults to installing [grafana-graphite-statsd 2.5.0](https://hub.docker.com/r/jlachowski/grafana-graphite-statsd) ([source](https://github.com/jlachowski/docker-grafana-graphite)).

## requirements

- dokku 0.4.0+
- docker 1.8.x

## installation

```shell
dokku plugin:install https://github.com/dokku/dokku-graphite-grafana.git graphite
```

## commands

```
graphite:create <name>            Create a graphite service with environment variables
graphite:destroy <name>           Delete the service and stop its container if there are no links left
graphite:expose <name> [port]     (WIP) Expose a graphite service on custom port if provided (random port otherwise)
graphite:info <name>              Print the connection information
graphite:link <name> <app>        Link the graphite service to the app
graphite:list                     List all graphite services
graphite:logs <name> [-t]         Print the most recent log(s) for this service
graphite:promote <name> <app>     Promote service <name> as STATSD_URL in <app>
graphite:restart <name>           Graceful shutdown and restart of the graphite service container
graphite:start <name>             Start a previously stopped graphite service
graphite:stop <name>              Stop a running graphite service
graphite:unexpose <name>          (WIP) Unexpose a previously exposed graphite service
graphite:unlink <name> <app>      Unlink the graphite service from the app
```

## usage

```shell
# create a graphite service named lolipop
dokku graphite:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# jlachowski/grafana-graphite-statsd image
export GRAPHITE_IMAGE="jlachowski/grafana-graphite-statsd"
export GRAPHITE_IMAGE_VERSION="2.5.0"

# create a graphite service
dokku graphite:create lolipop

# get connection information as follows
dokku graphite:info lolipop

# a graphite service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku graphite:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_GRAPHITE_LOLIPOP_PORT=tcp://172.17.0.1:8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP=udp://172.17.0.1:8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_PROTO=tcp
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_PORT=8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   STATSD_URL=statsd://dokku-graphite-lolipop:8125
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku graphite:link other_service playground

# since STATSD_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_STATSD_BLUE_URL=statsd://dokku-graphite-other_service:8125

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku graphite:promote other_service playground

# this will replace STATSD_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   STATSD_URL=statsd://dokku-graphite-other_service:8125
#   DOKKU_STATSD_BLUE_URL=statsd://dokku-graphite-other_service:8125
#   DOKKU_STATSD_SILVER_URL=statsd://dokku-graphite-lolipop:8125

# you can also unlink a graphite service
# NOTE: this will restart your app and unset related environment variables
dokku graphite:unlink lolipop playground

# you can tail logs for a particular service
dokku graphite:logs lolipop
dokku graphite:logs lolipop -t # to tail
```

## TODO:
- fix frontend port forwarding, so it's persistent on docker service resets, or ...
- provide nginx vhost config for each graphite service (grafena frontend) 
