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
at `tigerj/cups-airprint`. Simply running commands using this image will pull
the matching image architecture (e.g. `amd64`, `arm32v7`) based on the hosts
architecture. Hence, if you are on a **Raspberry Pi** the below commands will
work the same as if you were on a traditional `amd64` desktop/laptop computer.

# Getting Started
This section will give an overview of the essential options/arguments to pass
to docker to successfully run containers from the `tigerj/cups-airprint` docker
image.

## Run
To simply do a quick and dirty run of the cups/airprint container:
```
docker run
       --rm \
       --name=cups \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       -v $PWD/airprint_data/config:/config \
       -v $PWD/airprint_data/services:/services \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       tigerj/cups-airprint
```
Notice the `--rm` flag will ensure that when you terminate the container's
running process the container will be deleted

## Create
Creating a container is often better than out right running it:
```
docker create \
       --name=cups \
       --restart=always \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       -v $PWD/airprint_data/config:/config \
       -v $PWD/airprint_data/services:/services \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       tigerj/cups-airprint
```
Follow this with `docker start` and your cups/airprint printer is running:
```
docker start cups
```

### Parameters
* `--rm`: removes a container once it is stopped (see above)
* `--name`: gives the container a name making it easier to work with/on (e.g.
  `cups`)
* `--restart`: restart policy for how to handle restarts (e.g. `always` restart)
* `--net`: network to join (e.g. the `host` network)
* `-v $PWD/airprint_data/config:/config`: where the persistent printer configs
   will be stored
* `-v $PWD/airprint_data/services:/services`: where the Avahi service files will
   be generated
* `-e CUPSADMIN`: the CUPS admin user you want created
* `-e CUPSPASSWORD`: the password for the CUPS admin user
* `--device /dev/bus`: device mounted for interacting with USB printers
* `--device /dev/usb`: device mounted for interacting with USB printers

## Using
CUPS will be configurable at http://[diskstation]:631 using the
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
