# About
Modified copy of source code at:
https://github.com/quadportnick/docker-cups-airprint

# Intro
This Ubuntu-based Docker image runs a CUPS instance that is meant as an AirPrint
relay for printers that are already on the network but not AirPrint capable.
The local Avahi will be utilized for advertising the printers on the network.

This is also an excuse to dip my toes into GitHub/Docker more, so why not?
Hopefully someone else finds this useful.

# Multi-arch Image
The below commands reference a
[Docker Manifest List](https://docs.docker.com/engine/reference/commandline/manifest/)
at [`tigerj/cups-airprint`](https://hub.docker.com/r/tigerj/cups-airprint)
built using Docker's
[BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/).
Simply running commands using this image will pull
the matching image architecture (e.g. `amd64`, `arm32v7`, or `arm64`) based on
the hosts architecture. Hence, if you are on a **Raspberry Pi** the below
commands will work the same as if you were on a traditional `amd64`
desktop/laptop computer.**Note**: Because the image requires `ubuntu` as its base
image, there is currently no `arm32v6` architecture available. This means if your
target hardware is a **Raspberry Pi Zero** or similar `arm 6` architecture, this
image will not run.

# Getting Started
This section will give an overview of the essential options/arguments to pass
to docker to successfully run containers from the `tigerj/cups-airprint` docker
image.

## Run
To simply do a quick and dirty run of the cups/airprint container:
```
$ docker run
       -d \
       --name=cups \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       tigerj/cups-airprint
```
To stop the container simply run:
```
$ docker stop cups
```
To remove the conainer simply run:
```
$ docker rm cups
```
**WARNING**: Be aware that deleting the container (i.e. `cups` in the example)
will permanently delete the data that `docker volume` is storing for you.
If you want to permanently persist this data, see the `docker create` example
[below](#create). Continue reading the *Notes* section for more details about
Docker volumes

+ **Notes**: The `Dockerfile` explicitly sets volumes at `/config` and
`/services` (see
[these lines](https://github.com/RagingTiger/docker-cups-airprint/blob/2a30b6690a08262fb64375b74f07ab7b3f77ec4a/Dockerfile#L16-L17)).
 The necessary configurations done by the `docker container` will be
stored in those directories and will persist even if the container stops. Docker
will store the contents of these directories (located in the container) in
`/var/lib/docker/volumes` (see for reference
[Docker Volumes](https://docs.docker.com/storage/volumes/)).

## Create
Creating a container is often more desirable than directly running it:
```
$ docker create \
       --name=cups \
       --restart=always \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       -v ~/airprint_data/config:/config \
       -v ~/airprint_data/services:/services \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       tigerj/cups-airprint
```
Follow this with `docker start` and your cups/airprint printer is running:
```
$ docker start cups
```
To stop the container simply run:
```
$ docker stop cups
```
To remove the conainer simply run:
```
$ docker rm cups
```

+ **Notes**: As mentioned in the *Notes* subsection of the [Run](#run) section,
the `Dockerfile` explicitly declares two volumes at `/config` and `/services`
inside the container as mount points. Here we actually override the default
use of Docker's innate volume management system and declare our own path on the
host system to mount the two directories `/config` and `/services`. Why? Because
now if the container is deleted (for any number of reason ...) the data will
persist. Here we chose to mount the internal `/config` and `/services`
directories to `~/airprint_data/config` and `~/airprint_data/services`
respectively, but these could just as well be anywhere on your file system.

### Parameters
* `--name`: gives the container a name making it easier to work with/on (e.g.
  `cups`)
* `--restart`: restart policy for how to handle restarts (e.g. `always` restart)
* `--net`: network to join (e.g. the `host` network)
* `-v ~/airprint_data/config:/config`: where the persistent printer configs
   will be stored
* `-v ~/airprint_data/services:/services`: where the Avahi service files will
   be generated
* `-e CUPSADMIN`: the CUPS admin user you want created
* `-e CUPSPASSWORD`: the password for the CUPS admin user
* `--device /dev/bus`: device mounted for interacting with USB printers
* `--device /dev/usb`: device mounted for interacting with USB printers

## Using
CUPS will be configurable at http://localhost:631 using the
CUPSADMIN/CUPSPASSWORD when you do something administrative.

If the `/services` volume isn't mapping to `/etc/avahi/services` then you will
have to manually copy the .service files to that path at the command line.

## Notes
* CUPS doesn't write out `printers.conf` immediately when making changes even
though they're live in CUPS. Therefore it will take a few moments before the
services files update
* Don't stop the container immediately if you intend to have a persistent
configuration for this same reason

## ToDo
* Write common errors section
* Write CUPS printer setup section 
