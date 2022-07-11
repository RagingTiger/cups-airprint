# Archived
No further updates to the repository will be made at this time. Please see the _forks_ for any possible solutions to your specific use case and/or the 
author-suggested repo at: https://github.com/chuckcharlie/cups-avahi-airprint.

# <a name="toc"></a> Table of Contents
* [About](#about)
* [Intro](#intro)
* [Multi-arch Image](#multi-arch)
* [Getting Started](#start)
  + [Docker Run](#drun)
  + [Docker Create](#dcreate)
    + [Parameters](#dparams)
  + [Docker Compose](#dcompose)
  + [Docker Build](#dbuild)
* [Using](#using)
* [Notes](#notes)
* [Trouble Shooting](#trouble)
  + [Missing Printer Driver](#missing-driver)
  + [Driver Version](#driver-version)


# <a name="about"></a> [About](#toc)
Modified copy of source code at:
https://github.com/quadportnick/docker-cups-airprint

# <a name="intro"></a> [Intro](#toc)
This Ubuntu-based Docker image runs a CUPS instance that is meant as an AirPrint
relay for printers that are already on the network but not AirPrint capable.
The local Avahi will be utilized for advertising the printers on the network.

# <a name="multi-arch"></a> [Multi-arch Image](#toc)
The below commands reference a
[Docker Manifest List](https://docs.docker.com/engine/reference/commandline/manifest/)
at [`ghcr.io/ragingtiger/cups-airprint:master`](https://github.com/ragingtiger/cups-airprint/pkgs/container/cups-airprint)
built using Docker's
[BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/).
Simply running commands using this image will pull
the matching image architecture (e.g. `amd64`, `arm32v7`, or `arm64`) based on
the hosts architecture. Hence, if you are on a **Raspberry Pi** the below
commands will work the same as if you were on a traditional `amd64`
desktop/laptop computer. **Note**: Because the image requires `ubuntu` as its base
image, there is currently no `arm32v6` architecture available. This means if your
target hardware is a **Raspberry Pi Zero** or similar `arm 6` architecture, this
image will not run.

# <a name="start"></a> [Getting Started](#toc)
This section will give an overview of the essential options/arguments to pass
to docker to successfully run containers from the `ghcr.io/ragingtiger/cups-airprint:master`
docker image.

## <a name="drun"></a> [Docker Run](#toc)
To simply do a quick and dirty run of the cups/airprint container:
```
docker run \
       -d \
       --name=cups \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       ghcr.io/ragingtiger/cups-airprint:master
```
To stop the container simply run:
```
docker stop cups
```
To remove the conainer simply run:
```
docker rm cups
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

## <a name="dcreate"></a> [Docker Create](#toc)
Creating a container is often more desirable than directly running it:
```
docker create \
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
       ghcr.io/ragingtiger/cups-airprint:master
```
Follow this with `docker start` and your cups/airprint printer is running:
```
docker start cups
```
To stop the container simply run:
```
docker stop cups
```
To remove the conainer simply run:
```
docker rm cups
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

### <a name="dparams"></a> [Parameters](#toc)
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

## <a name="dcompose"></a> [Docker Compose](#toc)
If you don't want to type out these long **Docker** commands, you could
optionally use [docker-compose](https://docs.docker.com/compose/) to set up your
image. Just download the repo and run it like so:
```
git clone https://github.com/RagingTiger/docker-cups-airprint
cd docker-cups-airprint
docker-compose up
```
NOTE: This compose file is made with `USB` printers in mind and like the above
commands has `device` mounts for `USB` printers. If you don't have a `USB`
printer you may want to comment these out. Also the `config/services` data will
be saved to the users `$HOME` directory. Again you may want to edit this to
your own liking.

## <a name="dbuild"></a> [Docker Build](#toc)
If you would like to build the image yourself (locally), pull down the repo and
run the `docker build` command as follows:
```
git clone https://github.com/RagingTiger/docker-cups-airprint
cd docker-cups-airprint
docker build -t tigerj/cups-airprint .
```
Follow this with a [docker run](#drun) or [docker create](dcreate) to deploy
your container and your **cups-airprint** server is ready to be configured and
[used](#using).

## <a name="using"></a> [Using](#toc)
CUPS will be configurable at http://localhost:631 using the
CUPSADMIN/CUPSPASSWORD when you do something administrative.

If the `/services` volume isn't mapping to `/etc/avahi/services` then you will
have to manually copy the .service files to that path at the command line.

## <a name="notes"></a> [Notes](#toc)
* CUPS doesn't write out `printers.conf` immediately when making changes even
though they're live in CUPS. Therefore it will take a few moments before the
services files update
* Don't stop the container immediately if you intend to have a persistent
configuration for this same reason

## <a name="trouble"></a> [Trouble Shooting](#toc)
Here we are going to discuss the most **common problems** that users have when
trying to setup and configure their printer to work with the
**tigerj/cups-airprint** image.

### <a name="missing-driver"></a> [Missing Printer Driver](#toc)
As you might imagine this is **the most common** problem users have when setting
up their printers. While the **tigerj/cups-airprint** image possesses
**multiple printer drivers**, it most likely **does not** have every driver for
every printer. This issue can be resolved as follows:

+ Figure out what printer driver you need, open an issue about missing driver,
  necessary package containing said driver will be added to **Dockerfile**.

### <a name="driver-version"></a> [Driver Version](#toc)
Sometimes the right printer driver is installed in the **tigerj/cups-airprint**
Docker image, but the **version** is not current. This issue may require one of
two choices to resolve:

+ Download the **docker-cups-airprint** git repo and build a fresh image
  + This will pull the most recent versions of the printer driver from the package
    manager.

+ Download driver **DIRECTLY** from the manufacturer and add it to the image
  + If building a fresh image does not update the version of the driver, then
    you will need the most recent printer driver from the manufacturer.
